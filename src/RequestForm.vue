<template lang="html">
  <form @submit.stop.prevent="submit" v-if="selectedEntry" id="request-form">
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
        <v-text-field  class="pa-0" v-model.lazy.trim="currentRequest.params[parameter.name]" 
        :hint="parameter.example" persistent-hint
        :type="parameter.schema.type === 'string' ? 'text' : 'text'"></v-text-field>
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
      <v-textarea v-if="this.selectedEntry.method ==='post'" name="payload" v-model="currentRequest.body" id="test" rows="4"></v-textarea>
     <v-textarea v-else name="payload" v-model="currentRequest.putBody" id="test" rows="4"></v-textarea>
    </v-container>
  </form>
</template>

<script>
import Vue from "vue";
import patientBody from "@/assets/patient_sample_body.json";
import coverageBody from "@/assets/coverage_sample_body.json";
import locationBody from "@/assets/location_sample_body.json";
import practitionerBody  from "@/assets/practitioner_sample_body.json";
//import organizationBody  from "@/assets/organization_sample_body.json";
import ParametersTableVue from 'vue-openapi/src/ParametersTable.vue'
import MultipartForm from './MultipartForm.vue';
import stringify from "json-stringify-pretty-compact";
export default {
  components: {MultipartForm},
  props: ['selectedEntry', 'currentRequest'],
  mounted: function() {
    this.getDynamicExamples()
    this.$forceUpdate();
  },
  watch: {
    family: function () {
      this.currentRequest.params['family'] = this.family
      this.$forceUpdate();
    },
    claimResponseId: function(){
      this.currentRequest.params['resourceId'] = this.claimResponseId
      this.$forceUpdate();
    },
    beneficiary: function() {
      this.currentRequest.params['beneficiary'] = this.beneficiary
      this.$forceUpdate();
    },
    locationIdentifier: function() {
      this.currentRequest.params['identifier'] = this.locationIdentifier
      this.$forceUpdate();
    },
    practitionerIdentifier: function() {
      this.currentRequest.params['identifier'] = this.practitionerIdentifier
      this.$forceUpdate();
    },
    // organizationIdentifier: function() {
    //   this.currentRequest.params['identifier'] = this.organizationIdentifier
    //   this.$forceUpdate();
    // }
  },
  // updated: function() {
  //   if (this.family) {
  //     this.currentRequest.params['family'] = this.family
  //   } else if(this.claimResponseId){
  //     this.currentRequest.params['resourceId'] = this.claimResponseId
  //   } else if(this.beneficiary){
  //      this.currentRequest.params['beneficiary'] = this.beneficiary
  //   }
  // },
  data: () => ({
   family: '',
   beneficiary: '',
   claimResponseId: '',
   locationIdentifier: '',
   practitionerIdentifier: '',
   //organizationIdentifier: '',
   patientSampleBody: patientBody,
   coverageSampleBody: coverageBody,
   locationSampleBody: locationBody,
   practitionerSampleBody: practitionerBody,
   //organizationSampleBody: organizationBody,
  }),
  methods: {
    stringify,
    getFormData() {
      return this.$refs.multipartForm && this.$refs.multipartForm.getFormData()
    },
    getDynamicExamples(){
      var api = this.selectedEntry.path
      switch(true) {
        case api.includes('/Patient'):
          this.procesPatient();
          break;
        case api.includes('/Coverage'):
          this.procesCoverage();
          break;
        case api.includes('/ClaimResponse'):
          this.procesClaimResponse();
          break;
        case api.includes('/Location'):
          this.procesLocation();
          break;
        case api.includes('/Practitioner'):
          this.procesPractitioner();
          break;
        // case api.includes('/Organization'):
        //   this.procesOrganization();
        //   break;
        default:
          console.log("in default.......")
      }
    },

    // #Fire request to "Patient resource" get result , parse it and and set as example 
    procesPatient(){
      if (this.selectedEntry.parameters.length != 0) {
        if (this.selectedEntry.method ==='put') {
          this.currentRequest.putBody = JSON.stringify(this.patientSampleBody, null, 4);
        }
        return this.getPatientDynamicData();
      }if (this.selectedEntry.requestBody) {
        const dummyPatientBody = { ...this.patientSampleBody };
        delete dummyPatientBody.id
        this.currentRequest.body = JSON.stringify(dummyPatientBody, null, 4);
      }
    },

    getPatientDynamicData(){
      document.getElementById("overlay").style.display = "block";
      Vue.http({"url": "https://kong-dev.medecision.cloud/Patient/12207"}).then(
        response => {
          this.setPatientExamples(response)
        }).catch(e => {
          console.log(e);
          this.clearExamples();
          document.getElementById("overlay").style.display = "none";
      });
    },

    setPatientExamples(response){
      var parameters = this.selectedEntry.parameters;
      for (var key in parameters) {
        if ( ["_has","_page","_format"].includes( parameters[key].name)) {
           parameters[key]['example'] = this.setStaticExamples(parameters[key].name)
        }else{
          var res = this.lookup(response.body, parameters[key].name)
          var finalExample = this.parsePatient(res)
          parameters[key]['example'] = "For Example: "+ finalExample
          if (parameters[key].name === 'family') {
            this.family = finalExample
          }
        }
      }
      document.getElementById("overlay").style.display = "none";
    },

    parsePatient(res){
      if (Array.isArray(res)) {
        if (Array.isArray(res[1])) {
          if (res[0] === "identifier") {
            return this.deepLookUp(res, 'value')
          }else{
            return res[1][0]
          }
        }else{
          return res[1]
        }
      }
    },

    // #Fire request to "Coverage resource" get result , parse it and and set as example 
    procesCoverage(){
      if (this.selectedEntry.parameters.length != 0) {
        if (this.selectedEntry.method ==='put') {
          this.currentRequest.putBody = JSON.stringify(this.coverageSampleBody, null, 4);
        }
        return this.getCoveragetDynamicData();
      }if (this.selectedEntry.requestBody) {
        const dummyCoverageBody = { ...this.coverageSampleBody };
        delete dummyCoverageBody.id
        this.currentRequest.body = JSON.stringify(dummyCoverageBody, null, 4);
      }
    },

    getCoveragetDynamicData(){
      document.getElementById("overlay").style.display = "block";
      Vue.http({"url": "https://kong-dev.medecision.cloud/Coverage/1"}).then(
        response => {
          this.setCoverageExamples(response)
        }).catch(e => {
          console.log(e);
          this.clearExamples();
          document.getElementById("overlay").style.display = "none";
      });
    },

    setCoverageExamples(response){
      var parameters = this.selectedEntry.parameters;
      for (var key in parameters) {
        if ( ["_page"].includes( parameters[key].name)) {
           parameters[key]['example'] = this.setStaticExamples(parameters[key].name)
        }else{
          if (parameters[key].name === 'beneficiary:identifier') {
            var res = this.lookup(response.body, 'beneficiary')
            var finalExample = this.parseCoverage(res, 'beneficiary:identifier')
          }else if (parameters[key].name === 'beneficiary'){
            var res = this.lookup(response.body, parameters[key].name)
            var finalExample = this.parseCoverage(res);
            this.beneficiary = finalExample;
          }else {
            var res = this.lookup(response.body, parameters[key].name)
            var finalExample = this.parseCoverage(res);
          }
          if (res) {
            parameters[key]['example'] = "For Example: "+ finalExample;
          }else{
            parameters[key]['example'] = ""
          }
        }
      }
      document.getElementById("overlay").style.display = "none";
    },

    parseCoverage(res, extraKey){
      if (res!== null) {
        if (res[0] === 'beneficiary') {
          if (extraKey) {
            return this.deepLookUp(res, 'system')+ "|" + this.deepLookUp(res, 'value')
          }else{
            return this.deepLookUp(res, 'reference')
          }
        }else if (res[0] == 'type') {
          return this.deepLookUp(res, 'code')
        }else if(res[0] === 'period'){
          return this.deepLookUp(res, 'start')
        }else{
          return res[1]
        }
      }
    },

    // #Fire request to "ClaimResponse resource" get result , parse it and and set as example 
    procesClaimResponse(){
      if (this.selectedEntry.parameters.length != 0) {
          return this.getClaimResponseDynamicData();
      }
    },

    getClaimResponseDynamicData(){
      document.getElementById("overlay").style.display = "block";
      Vue.http({"url": "https://kong-dev.medecision.cloud/ClaimResponse/13094"}).then(response => {
          this.setClaimResponseExamples(response)
      }).catch(e => {
          console.log(e);
          this.clearExamples();
          document.getElementById("overlay").style.display = "none";
      });
    },

    setClaimResponseExamples(response){
      var parameters = this.selectedEntry.parameters;
      for (var key in parameters) {
          var res = this.lookup(response.body, parameters[key].name)
          var finalExample = this.parseClaimResponse(res)
          parameters[key]['example'] = "For Example: "+ finalExample
          this.claimResponseId = finalExample;
      }
       document.getElementById("overlay").style.display = "none";
    },

    parseClaimResponse(res){
      if (res!== null) {
        return res[1]
      }
    },

   // #Fire request to "Location resource" get result , parse it and and set as example 
    procesLocation(){
      if (this.selectedEntry.parameters.length != 0) {
        if (this.selectedEntry.method ==='put') {
          this.currentRequest.putBody = JSON.stringify(this.locationSampleBody, null, 4);
        }
        return this.getLocationDynamicData();
      }if (this.selectedEntry.requestBody) {
        const dummyLocationeBody = { ...this.locationSampleBody };
        delete dummyLocationeBody.id
        this.currentRequest.body = JSON.stringify(dummyLocationeBody, null, 4);
      }
    },

    getLocationDynamicData(){
      document.getElementById("overlay").style.display = "block";
      Vue.http({"url": "https://kong-dev.medecision.cloud/Location/28952"}).then(
        response => {
          this.setLocationExamples(response)
        }).catch(e => {
          console.log(e);
          this.clearExamples();
          document.getElementById("overlay").style.display = "none";
      });
    },

    setLocationExamples(response){
      var parameters = this.selectedEntry.parameters;
      for (var key in parameters) {
        var res = this.lookup(response.body, parameters[key].name)
        var finalExample = this.parseLocation(res)
        if ( ["_page"].includes( parameters[key].name)) {
          parameters[key]['example'] = this.setStaticExamples(parameters[key].name)
        }else{
          if (res) {
            if ((parameters[key].name === 'identifier')) {
              this.locationIdentifier = finalExample;
            }
            parameters[key]['example'] = "For Example: "+ finalExample;
          }else{
            parameters[key]['example'] = ""
          }
        }  
      }
      document.getElementById("overlay").style.display = "none";
    },

    parseLocation(res){
      if (res!== null) {
        if (Array.isArray(res[1])) {
          if (res[0] == "identifier") {
            return res[1][0]['system'] + "|" + this.deepLookUp(res, 'value')
          }else{
          return res[1][0]
          }
        }else{
          return res[1]
        }
      }
    },

   // #Fire request to "Practitioner resource" get result , parse it and and set as example 
    procesPractitioner(){
      if (this.selectedEntry.parameters.length != 0) {
        if (this.selectedEntry.method ==='put') {
          this.currentRequest.putBody = JSON.stringify(this.practitionerSampleBody, null, 4);
        }
        return this.getPractitionerDynamicData();
      }if (this.selectedEntry.requestBody) {
        const dummyPractitionerBody = { ...this.practitionerSampleBody };
        delete dummyPractitionerBody.id
        this.currentRequest.body = JSON.stringify(dummyPractitionerBody, null, 4);
      }
    },

    getPractitionerDynamicData(){
      document.getElementById("overlay").style.display = "block";
      Vue.http({"url": "https://kong-dev.medecision.cloud/Practitioner/55543"}).then(
        response => {
          this.setPractitionerExamples(response)
        }).catch(e => {
          console.log(e);
          this.clearExamples();
          document.getElementById("overlay").style.display = "none";
      });  
    },

    setPractitionerExamples(response){
      var parameters = this.selectedEntry.parameters;
      for (var key in parameters) {
        var res = this.lookup(response.body, parameters[key].name)
        var finalExample = this.parsePractitioner(res)
        if ( ["_page"].includes( parameters[key].name)) {
          parameters[key]['example'] = this.setStaticExamples(parameters[key].name)
        }else{
          if (res) {
            if ((parameters[key].name === 'identifier')) {
              this.practitionerIdentifier = finalExample;
            }
            parameters[key]['example'] = "For Example: "+ finalExample;
          }else{
            parameters[key]['example'] = ""
          }
        }  
      }
      document.getElementById("overlay").style.display = "none";
    },

    parsePractitioner(res){
      if (res!== null) {
        if (Array.isArray(res[1])) {
          if (res[0] == "identifier") {
            return res[1][0]['system'] + "|" + this.deepLookUp(res, 'value')
          }else{
            return res[1][0]
          }
        }else{
          return res[1]
        }
      }
    },

  // #Fire request to "organization resource" get result , parse it and and set as example 
    // procesOrganization(){
    //   if (this.selectedEntry.parameters.length != 0) {
    //     if (this.selectedEntry.method ==='put') {
    //       this.currentRequest.putBody = JSON.stringify(this.organizationSampleBody, null, 4);
    //     }
    //     return this.getOrganizationDynamicData();
    //   }if (this.selectedEntry.requestBody) {
    //     const dummyOrganizationBody = { ...this.organizationSampleBody };
    //     delete dummyOrganizationBody.id
    //     this.currentRequest.body = JSON.stringify(dummyOrganizationBody, null, 4);
    //   }
    // },
    // getOrganizationDynamicData(){
    //   document.getElementById("overlay").style.display = "block";
    //   Vue.http({"url": "https://kong-dev.medecision.cloud/Organization/50959"}).then(
    //     response => {
    //       this.setOrganizationExamples(response)
    //     }).catch(e => {
    //       console.log(e);
    //       this.clearExamples();
    //       document.getElementById("overlay").style.display = "none";
    //   });
    // },

    // setOrganizationExamples(response){
    //   var parameters = this.selectedEntry.parameters;
    //   for (var key in parameters) {
    //     var res = this.lookup(response.body, parameters[key].name)
    //     var finalExample = this.parseOrganisation(res)
    //     if ( ["_page"].includes( parameters[key].name)) {
    //       parameters[key]['example'] = this.setStaticExamples(parameters[key].name)
    //     }else{
    //       if (res) {
    //         if ((parameters[key].name === 'identifier')) {
    //           this.organizationIdentifier = finalExample;
    //         }
    //         parameters[key]['example'] = "For Example: "+ finalExample;
    //       }else{
    //         parameters[key]['example'] = ""
    //       }
    //     }  
    //   }
    //   document.getElementById("overlay").style.display = "none";
    // },

    // parseOrganisation(res){
    //   if (res!== null) {
    //     if (Array.isArray(res[1])) {
    //       if (res[0] == "identifier") {
    //         return res[1][0]['system'] + "|" + this.deepLookUp(res, 'value')
    //       }if (res[0] == "type"){
    //         return this.deepLookUp(res, 'code')
    //       }else{
    //         return res[1][0]
    //       }
    //     }else{
    //       return res[1]
    //     }
    //   }
    // },

  // Common method for Parsing Response hash(Lookup, DeepLookup) and mapping missing keys(mapKey)
    clearExamples(){
      this.currentRequest.params = {}
      var parameters = this.selectedEntry.parameters;
      for (var key in parameters) {
        parameters[key]['example'] = "";
      }
    },

    deepLookUp(res, value){
      var _exp = this.lookup(res[1], value)
        if (_exp) {
          return _exp[1]
        }else{
          return 'No data'
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
      }else if(k === 'subscriberid'){
        k = 'subscriberId'
      }else if(k === 'address-line1'){
        k = 'line'
      }else if(k === 'address-city'){
        k ='city'
      }else if(k === 'address-state'){
        k ='state'
      }else if(k === 'address-postalcode'){
        k ='postalCode'
      }else if(k === 'address-country'){
        k ='country'
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





