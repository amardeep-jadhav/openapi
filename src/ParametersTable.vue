<template lang="html">
  <v-simple-table class="pl-4 pb-5" v-if="(selectedEntry.parameters && selectedEntry.parameters.length) || selectedEntry.requestBody">
    <thead class="stone-gray">
      <tr>
        <th class="text-left tableHeading white--text text-h5">Parameter</th>
        <th class="text-left tableHeading white--text text-h5">Mandatory</th>
        <th class="text-left tableHeading white--text text-h5">Description</th>
        <th class="text-left tableHeading white--text text-h5">Type</th>
        <!-- <th>Location</th>
        <th>Required</th>
        <th>Examples</th> -->
      </tr>
    </thead>

    <tbody>
      <tr v-if="selectedEntry.requestBody">
        <td>Payload</td>
        <td>Yes</td>
        <td>Request body</td>
        <td v-if="!selectedEntry.requestBody.content"></td>
        <td v-if="selectedEntry.requestBody.content">
          <!-- <md-select v-model="selectedEntry.requestBody.selectedType">
            <md-option v-for="contentType in Object.keys(selectedEntry.requestBody.content)" :key="contentType" :value="contentType">{{contentType}}</md-option>
          </md-select> -->
          <v-select v-model="selectedEntry.requestBody.selectedType"
          :items="Object.keys(selectedEntry.requestBody.content)"
        >
        </v-select>
          
        </td>
        <!-- <td v-if="!selectedEntry.requestBody.content || !selectedEntry.requestBody.content[selectedEntry.requestBody.selectedType].schema"></td>
        <td v-if="selectedEntry.requestBody.content && selectedEntry.requestBody.content[selectedEntry.requestBody.selectedType].schema" style="align-items: left;">
          <v-icon class="md-accent" @click.native="openSchemaDialog(selectedEntry.requestBody.content[selectedEntry.requestBody.selectedType].schema)" style="cursor:pointer" title="View schema">open_in_new</v-icon>
        </td>
        <td>body</td>
        <td>
          <v-checkbox v-model="selectedEntry.requestBody.required" disabled></v-checkbox>
        </td>
        <td>
          <v-icon class="md-accent" v-if="examples(selectedEntry.requestBody.content[selectedEntry.requestBody.selectedType])" @click.native="openExamplesDialog(examples(selectedEntry.requestBody.content[selectedEntry.requestBody.selectedType]))" style="cursor:pointer">open_in_new</v-icon>
        </td> -->
      </tr>


      <tr v-for="(parameter, i) in selectedEntry.parameters" :key="i">
        <td>{{parameter.name}}</td>
        <td>{{parameter.required}}</td>
        <td v-html="marked(parameter.description || '')"></td>
        <td v-if="parameter.schema.type !== 'array'">{{parameter.schema.type}}</td>
        <td v-if="parameter.schema.type === 'array'">{{parameter.schema.items.type}} array</td>
         <!--<td v-if="parameter.schema.type !== 'array' && parameter.schema.enum">{{parameter.schema.enum.join(', ')}}</td>
        <td v-if="parameter.schema.type !== 'array' && !parameter.schema.enum"></td>
        <td v-if="parameter.schema.type === 'array'">
          <div style="overflow-y:scroll;max-height:200px;">{{(parameter.schema.items.enum || []).join(', ')}}</div>
        </td>
        <td>{{parameter.in}}</td>
        <td>
          <v-checkbox v-model="parameter.required" disabled></v-checkbox>
        </td>
        <td>
          <v-icon class="md-accent" v-if="examples(parameter)" @click.native="openExamplesDialog(examples(parameter))" style="cursor:pointer">open_in_new</v-icon>
        </td> -->
      </tr>
    </tbody>
  </v-simple-table>
</template>

<script>
import marked from 'marked'

export default {
  props: [ 'selectedEntry', 'openSchemaDialog', 'openExamplesDialog' ],
  methods: {
    marked,
    examples(content) {
      if (content.example) return {'Example': {summary: 'Default example', value: content.example}}
      if (content.examples) return content.examples
      if (content.schema && content.schema.example) return {'Example': {summary: 'Schema example', value: content.schema.example}}
      return null
    }
  }
}
</script>

<style lang="css">
p{
  margin-bottom: 0px;
}
</style>
