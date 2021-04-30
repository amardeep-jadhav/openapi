<template lang="html">
  <v-simple-table >
    <thead>
      <tr>
        <th>Name</th>
        <th>Type</th>
        <th>Location</th>
      </tr>
    </thead>

    <tbody>
      <tr v-for="(security, i) in selectedEntry.security" :key="i">
        <td>{{security.scheme.name}}</td>
        <td>{{security.scheme.type}}</td>
        <td>{{security.scheme.in}}</td>
      </tr>
    </tbody>
  </v-simple-table >
</template>

<script>
export default {
  props: [ 'selectedEntry' ]
}
</script>

<style lang="css">
</style>
