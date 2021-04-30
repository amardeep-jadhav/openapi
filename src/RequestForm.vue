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
    <div v-for="(parameter, i) in selectedEntry.parameters" :key="i">
      <v-container class="pa-0 pt-4" v-if="(parameter.schema.type === 'string' || parameter.schema.type === 'integer' || parameter.schema.type === 'number') && !parameter.schema.enum">
        <label class="text-h5 d-inline-block grape--text">{{parameter.name}}</label>
        <v-text-field  class="pa-0" v-model="currentRequest.params[parameter.name]" :type="parameter.schema.type === 'string' ? 'text' : 'number'"></v-text-field>
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

      <v-checkbox v-if="parameter.schema.type === 'boolean'" v-model="currentRequest.params[parameter.name]">{{parameter.name}}</v-checkbox>

    </div>

    <!-- <v-subheader v-if="selectedEntry.requestBody">
      Body
    </v-subheader> -->
    <multipart-form ref="multipartForm" v-if="selectedEntry.requestBody && selectedEntry.requestBody.selectedType === 'multipart/form-data'" :request-body="selectedEntry.requestBody" />
    <v-container v-else-if="selectedEntry.requestBody">
      <label class="text-h5 d-inline-block grape--text" for="payload">Payload ({{selectedEntry.requestBody.selectedType}})</label>
      <v-textarea name="payload" v-model="currentRequest.body" id="test"></v-textarea>
    </v-container>
  </form>
</template>

<script>
import MultipartForm from './MultipartForm.vue';
import stringify from "json-stringify-pretty-compact";
export default {
  components: {MultipartForm},
  props: ['selectedEntry', 'currentRequest'],
  methods: {
    stringify,
    getFormData() {
      return this.$refs.multipartForm && this.$refs.multipartForm.getFormData()
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
.container .v-input >>> .v-input__slot{
  margin-bottom: 0px !important;
}
.container .v-input >>> .v-text-field__details{
  min-height: 0px!important;
  height:0;
}
.container  >>> .v-input input {
    max-height: 20px;
    padding: 4px;
}
</style>
