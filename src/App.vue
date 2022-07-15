<template>
  <div class="hello">
 
    <div class = "input">
    <input @change="addToBody($event)"/>
   </div>

   <div class = "output">
    <textarea id="outputText" class="output" v-model="curlOutput"></textarea>
    </div>
     <div class = "output">
      <div>
        <input @change="addFlags($event)" placeholder="-k"/>
      </div>
       <div>
        <input @change="addMethod($event)" placeholder="Method GET/POST"/>
      </div>
       <div>
        <input @change="addHost($event)" placeholder="hostname"/>
      </div>
      <div>
        <input @change="addVariables($event)" placeholder="variables"/>
      </div>

      <div v-for="item in headerCount" :key="item.length">
        <input @change="addHeaderName($event)" placeholder="header name"/>
        <input @change="addHeaderValue($event)" placeholder="value"/>
      </div>
      <button v-on:click="newHeader">+</button>
     <button v-on:click="removeHeader">-</button>
    </div>
    <button @click="convert">Convert</button>

  </div>
</template>

<script>

export default {
  name: 'App',

  data(){
    return{
      flags: '-k',
      method: 'post',
      host: 'https://someplace.com:8443/gql/payments',
      queryType: '',
      gqlInputBody: 'query {pets {name petType}}',
      gqlOutputBody:'',
      hasVariables : false,
      variables: '',
      headerNames: ['Authorization','content-type'],
      headerValues: ['bearer skdasfsdmfsf3425','application/json'],
      curlOutput: '',
      headerCount: 3,
      headerOutput: ''
    }
  },
  methods: {
    addFlags:function(e){
      this.flags = e.target.value;
    },
    addMethod:function(e){
      this.method = e.target.value.toUpperCase();
    },
    addHost:function(e){
      this.host = e.target.value;
    },
    addToBody:function(e){
      this.gqlInputBody+=e.target.value; 
    },
    newHeader: function(){
      this.headerCount++;
    },
    removeHeader:function(){
      this.headerCount--;
    },
    addHeaderName: function(e){
      this.headerNames.push(e.target.value)
    },
    addHeaderValue: function(e){
      this.headerValues.push(e.target.value)
    },
    addVariables: function(e){
      this.variables=(e.target.value)
      console.log(this.variables)
    },
    formatVariables: function(){
      console.log('inside format variables',this.variables)
      this.variables = JSON.stringify(this.variables)
      this.variables = this.variables.replace('{','')
      this.variables = this.variables.split('}')
     
      this.variables.forEach(item => item = item+'"')
      console.log(this.variables)
      
    },
    setBody: function () {
      console.log(this.gqlInputBody)
        if(this.gqlInputBody==''){
          return
        }
       this.gqlOutputBody = "-d"+"'"+"{"+'"'+"query"+'"'+":"+'"'+`${this.gqlInputBody}`+'"'+"}"+"'"+` \\`
      
    },
    setHeaders: function(){
      console.log(this.headerNames,this.headerValues)
       this.headerNames.forEach( (header, index) => {
         this.headerOutput += '-H '+'"'+`${header}: ${this.headerValues[index]}`+`" \\ \n`
         index ++;
      })
    },
    convert: function(){
      this.setHeaders();
      this.setBody();
      
      if(this.variables.length > 1){  
        this.formatVariables()
        this.gqlOutputBody+='"'+"variables"+'"'+":"+"{"+this.variables+"}"  
      }
      this.curlOutput = `curl --request ${this.flags} ${this.method} ${this.host} \\ \n${this.headerOutput}${this.gqlOutputBody}`;
      this.gqlInputBody = '',
      this.gqlOutputBody = ''
      this.queryType = ''
      this.variables = ''
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.output{
  width: 80%;
  height:7em;
}
</style>
