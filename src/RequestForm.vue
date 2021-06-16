<template lang="html">
    
  <form novalidate @submit.stop.prevent="submit" v-if="selectedEntry" id="request-form">
    <h6 class="text-h6 d-inline-block" v-if="selectedEntry.security && selectedEntry.security.filter(s => s.scheme.in !== 'cookie').length">
      Security
    </h6>
    <div v-for="(security, i) in selectedEntry.security.filter(s => s.scheme.in !== 'cookie')" :key="i">
      <v-container>
        <label>{{security.scheme.name}}</label>
        <v-text-field v-model="currentRequest.security[security.scheme.name]" type="string"></v-text-field>
      </v-container>
    </div>

    <h5 class="text-h5 d-inline-block fuchsia--text" v-if="selectedEntry.parameters && selectedEntry.parameters.length">
      Parameters
    </h5>
    <div v-for="(parameter, i) in selectedEntry.parameters" :key="parameter.example">
      <v-container class="pa-0 pt-4" v-if="(parameter.schema.type === 'string' || parameter.schema.type === 'integer' || parameter.schema.type === 'number') && !parameter.schema.enum">
        <label class="text-h5 d-inline-block grape--text">{{parameter.name}}</label>
        
        <v-text-field  class="pa-0" v-model.lazy="currentRequest.params[parameter.name]" 
        :hint="parameter.example" persistent-hint
        :type="parameter.schema.type === 'string' ? 'text' : 'number'"></v-text-field>
      </v-container>

      <v-container v-if="parameter.schema.enum">
        <label class="text-h5 d-inline-block grape--text">{{parameter.name}}</label>
        <!-- <md-select v-model="currentRequest.params[parameter.name]">
          <md-option v-for="val in parameter.schema.enum" :key="val" :value="val">{{val}}</md-option>
        </md-select> -->
        <v-select v-model="currentRequest.params[parameter.name]" 
        :items="parameter.schema.enum">
        </v-select>
      </v-container>

      <v-container v-if="parameter.schema.type === 'array' && parameter.schema.items.enum">
        <label class="text-h5 d-inline-block grape--text">{{parameter.name}}</label>
        <!-- <md-select v-model="currentRequest.params[parameter.name]" multiple>
          <md-option v-for="val in parameter.schema.items.enum" :key="val" :value="val">{{val}}</md-option>
        </md-select> -->
         <v-select  v-model="currentRequest.params[parameter.name]" multiple
        :items="parameter.schema.items.enum">
        </v-select>
      </v-container>

      <v-chip v-model="currentRequest.params[parameter.name]" :v-text-field-placeholder="parameter.name" :v-text-field-type="parameter.schema.items.type" v-if="parameter.schema.type === 'array' && !parameter.schema.items.enum">
        <template slot-scope="chip">{{ chip.value }}</template>
      </v-chip>

      <v-container class="pa-0 pt-4" v-if="parameter.schema.type === 'boolean'">
        <label class="text-h5 d-inline-block grape--text">{{parameter.name}}</label>
        <v-checkbox v-if="parameter.schema.type === 'boolean'" :hint="parameter.example" v-model="currentRequest.params[parameter.name]">{{parameter.name}}</v-checkbox>
      </v-container>
    </div>

    <!-- <v-subheader v-if="selectedEntry.requestBody">
      Body
    </v-subheader> -->
    <multipart-form ref="multipartForm" v-if="selectedEntry.requestBody && selectedEntry.requestBody.selectedType === 'multipart/form-data'" :request-body="selectedEntry.requestBody" />
    <v-container v-else-if="selectedEntry.requestBody">
      <label class="text-h5 d-inline-block grape--text" for="payload">Payload ({{selectedEntry.requestBody.selectedType}})</label>
      <v-textarea name="payload" v-model="currentRequest.body" id="test" rows="4"></v-textarea>
    </v-container>
  </form>
</template>

<script>
import Vue from "vue";
import ParametersTableVue from 'vue-openapi/src/ParametersTable.vue'
import MultipartForm from './MultipartForm.vue';
import stringify from "json-stringify-pretty-compact";
export default {
  components: {MultipartForm},
  props: ['selectedEntry', 'currentRequest'],
  created: function() {
    this.getDynamicExamples()
    this.$forceUpdate();
  },
  watch: {
    family: function () {
      this.currentRequest.params['family'] = this.family
      this.$forceUpdate();
    }
  },
  updated: function() {
    if (this.family) {
      this.currentRequest.params['family'] = this.family
    }
  },
  data: () => ({
   family: ''
  }),
  methods: {
    stringify,
    getFormData() {
      return this.$refs.multipartForm && this.$refs.multipartForm.getFormData()
    },
    getDynamicExamples(){
      if (this.selectedEntry.path.includes('/Patient')) {
        if (this.selectedEntry.parameters.length != 0) {
          return this.getPatientDynamicData();
        }
      }else{
        return; 
      }
    },
    getPatientDynamicData(){
      Vue.http({"url": "https://kong-dev.medecision.cloud/Patient?family=1002L"}).then(
        response => {
          this.setPatientExamples(response)
        }
      );
    },
    setPatientExamples(response){
      var parameters = this.selectedEntry.parameters;
      for (var key in parameters) {
        if ( ["_has","_page","_format"].includes( parameters[key].name)) {
           parameters[key]['example'] = this.setStaticExamples(parameters[key].name)
        }else{
          var res = this.lookup(response.body.entry, parameters[key].name)
          var finalExample = this.parseExample(res)
          parameters[key]['example'] = "For Example: "+ finalExample
          if (parameters[key].name === 'family') {
            this.family = finalExample
          }
        }
      }
    },
    parseExample(res){
      if (Array.isArray(res)) {
        if (Array.isArray(res[1])) {
          if (res[0] === "identifier") {
            return res[1][0].value
          }else{
            return res[1][0]
          }
        }else{
          return res[1]
        }
      }
    },
    lookup(obj, k) {
      k = this.mapKey(k)
      for (var key in obj) {
        var value = obj[key];
        if (k == key) {
          return [k, value];
        }
        if (typeof(value) === "object" && !Array.isArray(value)) {
          var y = this.lookup(value, k);
          if (y && y[0] == k) return y;
        }
        if (Array.isArray(value)) {
          for (var i = 0; i < value.length; ++i) {
            var x = this.lookup(value[i], k);
            if (x && x[0] == k) return x;
          }
        }
      }
      return null;
    },
    mapKey(k){
      if (k === 'birthdate') {
        k = 'birthDate'
      }else if(k === 'resourceId'){
        k = 'id'
      }
      return k;
    },
    setStaticExamples(name){
      if (name === "_format" ) {
        return "For Example : JSON";
      } else if(name === "_page") {
        return "For Example : 0 ";
      }
    }
  }
}
</script>

<style lang="css" scoped>
.openapi #request-form {
    padding: 0;
    height:200px;
overflow-y:auto;
}
textarea {
    font-family: 'museo-sans';
    font-size: 1rem;
}
/* .container .v-input >>> .v-input__slot{
  margin-bottom: 0px !important;
} */
/* .container .v-input >>> .v-text-field__details{
  min-height: 0px!important;
  height:0;
} */
.container  >>> .v-input input {
    max-height: 20px;
    padding: 4px;
}
</style>
