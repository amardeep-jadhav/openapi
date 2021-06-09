<template lang="html">
  <!-- <md-layout md-column class="response-display">
    <span>{{entry.method.toUpperCase()}} {{response.url}}&nbsp;&nbsp;&nbsp;&nbsp;<v-chip :class="response.ok ? 'pink':'warning'">{{response.status}} {{response.statusText}}</v-chip></span>
    <br>

    <md-list class="md-dense">
      <md-list-item>
        <md-subheader style="padding-left:0px;">Headers</md-subheader>
        <md-list-expand>
          <md-list>
            <md-list-item v-for="header in Object.keys(response.headers.map)">{{header}}: {{Array.isArray(response.headers.map[header]) ? response.headers.map[header].join(',') : response.headers.map[header]}}</md-list-item>
          </md-list>
        </md-list-expand>
      </md-list-item>
    </md-list>
    <md-subheader>Body</md-subheader>
    <pre>{{formattedBody}}</pre>
  </md-layout> -->
 <v-col class="response-display px-4 py-0">
    <span class="text-wrap text-h5" style="word-break:break-all"><label :id="setColor(entry.method)" class="">{{entry.method.toUpperCase()}}</label> {{response.url}}&nbsp;&nbsp;&nbsp;&nbsp;<v-chip :id="response.ok ? 'post_Bgcolor':'put_Bgcolor'" class="font-weight-bold white--text">{{response.status}} {{response.statusText}}</v-chip></span>
    <!-- <v-list dense>
      <v-list-item>        
        <v-expansion-panels>
          <v-expansion-panel>
            <v-expansion-panel-header>Headers</v-expansion-panel-header>
          <v-expansion-panel-content v-for="header in Object.keys(response.headers.map)">
            {{header}}: {{Array.isArray(response.headers.map[header]) ? response.headers.map[header].join(',') : response.headers.map[header]}}
          </v-expansion-panel-content>
          </v-expansion-panel>
        </v-expansion-panels>
      </v-list-item>
    </v-list> -->

      <!-- <v-subheader>Response Sample</v-subheader> -->   
      <!-- <pre class="MS_300_12px accent--text text--darken-4">{{formattedBody}}</pre> -->
      <v-container>
      <vue-json-pretty :data= this.formattedBody :showLength="true"></vue-json-pretty>
      </v-container>
  </v-col>


</template>

<script>
import stringify from 'json-stringify-pretty-compact'
import VueJsonPretty from 'vue-json-pretty'
import 'vue-json-pretty/lib/styles.css';
export default {
  components: { VueJsonPretty},
  props: ['response', 'entry'],
  computed: {
    formattedBody() {
      let res
      try {
        res = stringify(this.response.body, null, 2)
        if (typeof res === 'string') res = this.response.body
      } catch (err) {
        res = this.response.body
      }
      return res
    }
  },
  methods: {  
    select(entry) {
    //   this.reset(entry);
    //   this.selectedEntry = entry;
      this.$emit("onSelectEntry", entry);
    },   
    setColor(method_name){
      return method_name+"_color"
    }
  }
}
</script>

<style lang="css">
.response-display pre {
  white-space: pre-wrap;
}
.response-display .container{
  min-height: 50px;
  max-height:200px;
  overflow-y:auto;
}
.vjs-tree {
    font-family: 'museo-sans';
    font-size: 1rem;
}
.v-dialog .response-display .container {
    min-height: 50px;
    max-height: none;
    overflow-y: auto;
}
</style>
