<template lang="html">
  <v-simple-table>
    <thead>
      <tr>
        <th>HTTP Code</th>
        <th>Response</th>
        <th>Type</th>
        <th>Schema</th>
        <th>Headers</th>
        <th>Examples</th>
      </tr>
    </thead>

    <tbody>
      <tr v-for="(response, code) in selectedEntry.responses" :key="code">
        <td>{{code}} </td>
        <td v-html="marked(response.description || '')"></td>

        <td v-if="response.content">
          <v-select v-model="response.selectedType">
            <md-option v-for="(value, content) in response.content" :key="content" :value="content">{{content}}</md-option>
          </v-select>
        </td>
        <td v-else />

        <td v-if="response.content && response.content[response.selectedType].schema">
          <v-icon color="primary" @click.native="openSchemaDialog(response.content[response.selectedType].schema)" style="cursor:pointer">open_in_new</v-icon>
        </td>
        <td v-else />

        <td v-if="response.headers">
          <v-icon color="primary" @click.native="openFieldsDialog(response.headers)" style="cursor:pointer">open_in_new</v-icon>
        </td>
        <td v-else />

        <td>
          <v-icon color="primary" v-if="examples(response)" @click.native="openExamplesDialog(examples(response))" style="cursor:pointer">open_in_new</v-icon>
        </td>

      </tr>
    </tbody>
  </v-simple-table>
</template>

<script>
import marked from 'marked'

export default {
  props: [ 'selectedEntry', 'openSchemaDialog', 'openExamplesDialog', 'openFieldsDialog'],
  methods: {
    marked,
    examples(response) {
      const content = response.content && response.content[response.selectedType]
      if (content) {
        if (content.example) return {'Example': {summary: 'Default example', value: content.example}}
        if (content.examples) return content.examples
        if (content.schema && content.schema.example) return {'Example': {summary: 'Schema example', value: content.schema.example}}
      }
      return null
    }
  }
}
</script>

<style lang="css">
</style>
