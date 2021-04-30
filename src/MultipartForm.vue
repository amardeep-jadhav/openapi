<template lang="html">
  <v-row>
    <v-col cols="12" sm="12" md="12" v-for="key in Object.keys(properties)" :key="key">
      <!-- File upload -->
      <v-container v-if="properties[key].type === 'string' && properties[key].format === 'binary'">
        <label>{{properties[key].title || key}}</label>
        <v-file-input
      chips
      multiple @selected="files => onFileUpload(key, files)"></v-file-input>
      </v-container>

      <!-- simple enum -->
      <v-container v-else-if="properties[key].type === 'array' && properties[key].items.enum">
        <label>{{properties[key].title || key}}</label>
        <!-- <md-select v-model="model[key]" multiple>
          <md-option v-for="val in properties[key].items.enum" :key="val" :value="val">{{val}}</md-option>
        </md-select> -->
         <v-select v-model="model[key]" multiple
        :items="properties[key].items.enum">
        </v-select>
      </v-container>

      <!-- multi enum -->
      <v-container v-else-if="properties[key].enum">
        <label>{{properties[key].title || key}}</label>
        <!-- <md-select v-model="model[key]">
          <md-option v-for="val in properties[key].enum" :key="val" :value="val">{{val}}</md-option>
        </md-select> -->
         <v-select v-model="model[key]"
        :items="properties[key].enum">
        </v-select>
      </v-container>

      <!-- boolean -->
      <v-checkbox v-else-if="properties[key].type === 'boolean'" v-model="model[key]">{{properties[key].title || key}}</v-checkbox>

      <!-- generic input -->
      <v-container v-else-if="properties[key].type === 'string' || properties[key].type === 'integer' || properties[key].type === 'number'">
        <label>{{properties[key].title || key}}</label>
        <v-text-field v-model="model[key]" :type="properties[key].type === 'string' ? 'text' : 'number'"></v-text-field>
      </v-container>

      <!-- objects in textarea -->
      <v-container v-else>
        <label>{{properties[key].title || key}}</label>
        <v-textarea v-model="model[key]"></v-textarea>
      </v-container>
    </v-col>
  </v-row>
</template>

<script>
const FormData = require('form-data')

export default {
  props: ['requestBody'],
  data() {
    return {model: {}}
  },
  computed: {
    properties() {
      return this.requestBody.content['multipart/form-data'].schema.properties
    }
  },
  mounted() {
    Object.keys(this.properties).forEach(key => {
      this.$set(this.model, key, null)
    })
  },
  methods: {
    onFileUpload(key, files) {
      this.model[key] = files[0]
    },
    getFormData() {
      const form = new FormData()
      Object.keys(this.properties)
        .filter(key => !!this.model[key])
        .forEach(key => {
          if (['object', 'array'].includes(this.properties[key].type)) {
            if (this.model[key].trim() !== '') form.append(key, this.model[key].trim())
          } else {
            form.append(key, this.model[key])
          }
        })
      return form
    }
  }
}
</script>

<style lang="css">
</style>
