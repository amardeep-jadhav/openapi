<template lang="html">
  <form novalidate @submit.stop.prevent="submit" v-if="selectedEntry" id="request-form" class="request-form">
    <v-container v-if="selectedEntry.requestBody && this.selectedEntry.requestBody.selectedType =='text/xml'">
      <pre> 
        {{this.formatXML(this.selectedEntry.requestBody.content[selectedEntry.requestBody.selectedType].schema.example)}}
      </pre>
    </v-container>
    <v-container v-else>
      <vue-json-pretty
        :path="'res'"
        :data= this.getValidJson(selectedEntry.requestBody.content[selectedEntry.requestBody.selectedType].schema)
        :showLength="true"
        @click="populateSampleData(getValidJson(selectedEntry.requestBody.content[selectedEntry.requestBody.selectedType].schema))"
        > 
      </vue-json-pretty>
    </v-container>
  </form>
</template>

<script>
import stringify from "json-stringify-pretty-compact";
import VueJsonPretty from 'vue-json-pretty'
import 'vue-json-pretty/lib/styles.css';
export default {
  components: { VueJsonPretty},
  props: ['selectedEntry', 'currentRequest'],
  filters: {
  },
  methods: {
    formatXML(data){
      var format = require('xml-formatter');
      return format(data);
    },
    copyXMLDtaa(){
      let data = JSON.parse(JSON.stringify(this.selectedEntry.requestBody.content))["text/xml"]["example"] 
      this.currentRequest.body = this.formatXML(data)
    },
    stringify,
    // getBodySample(){
    //  let json = this.selectedEntry.requestBody;
    //   return json;
    // },
    populateSampleData(data){
      this.currentRequest.body = JSON.stringify(data, null, 4);
    },
    getValidJson(data){
      this.currentRequest.body = null;
      let json = {}
      if (data.properties) {
         Object.keys(data.properties)
        .forEach(key => {
          if (['object', 'array'].includes(data.properties[key].type)) {
            json[key] = this.formatJson(data.properties[key])
          } else {
            json[key] = data.properties[key].type
          }
        })
        return json
      }else{
         Object.keys(data.items.properties)
        .forEach(key => {
          if (['object', 'array'].includes(data.items.properties[key].type)) {
            json[key] = this.formatJson(data.items.properties[key])
          } else {
            json[key] = data.items.properties[key].type
          }
        })
        return [json]
      }
    },
    formatJson(data){
      let json = {}
      if (data.properties == undefined) {
        if (data.items.properties) {
          Object.keys(data.items.properties)
          .forEach(key => {
            if (['object', 'array'].includes(data.items.properties[key].type)) {
              json[key] = this.formatJson(data.items.properties[key])
            } else {
              json[key] = data.items.properties[key].type
            }
          })
        }else{
          return [data.items.type]  
        }
      }else{
        Object.keys(data.properties)
        .forEach(key => {
          if (['object', 'array'].includes(data.properties[key].type)) {
            json[key] = this.formatJson(data.properties[key])
          } else {
            json[key] = data.properties[key].type
          }
        })
      }
    return json
    }
  }
}
</script>

<style lang="css" scoped>
.openapi #request-form {
    padding: 0;
}
pre{ 
  white-space: pre-wrap;
/* word-wrap: break-word; */
overflow-x: auto;
}
.request-form .container{
  min-height: 50px;
  max-height:200px;
  overflow-y:auto;
}
.vjs-tree {
    font-family: 'museo-sans';
    font-size: 1rem;
}
.v-dialog .request-form .container {
    min-height: 50px;
    max-height: none;
    overflow-y: auto;
}
</style>
