  
<template>
  <v-expansion-panels class="md-dense stone-gray-dark" ref="menu">
    <v-expansion-panel class="stone-gray-dark py-4 px-2 contentOverflow"
      v-for="(entries, tag) in tags" v-if="tag === apiVerb"
      :key="tag"
    >
      <v-expansion-panel class="stone-gray-dark pl-0 pt-1 pb-n2 clearfix"
        v-for="(entry, i) in modifiedTags"
        :key="i"
        @click.native
        style="cursor: pointer;"
      >
      <h5 class="px-4 py-1 text-body-2 float-left d-block clsEntryMethod text-center" :id="setColor(entry.method)">{{
          entry.method.toUpperCase()
        }}</h5>
        <a class="px-4 py-1 text-subtitle-2 d-block text-truncate clsEntryPath off-white--text"
          :class="{
            'text-subtitle-2': selectedEntry === entry,
          }"
          v-html="entry.summary"       
          :title="entry.summary"
          v-bind:href="'#'+ entry.summary"
        ></a>&nbsp;
      </v-expansion-panel>
    </v-expansion-panel>
  </v-expansion-panels>
</template>
<script>
export default {
  components: {
 
  },
  props: ["tags", "selectedEntry"],
  data: () => ({
    modifiedTags: null,
    apiVerb: ""
  }),
  mounted: function () {
    if (this.$refs.menu.$children.length)
    this.$refs.menu.$children[0].toggleExpandList();
  },
  watch: {
    tags: function(){
      this.apiVerb = window.location.href.split('?')[1].split("=")[1]
      if (this.apiVerb.includes("#")){
        this.apiVerb = this.apiVerb.match(/.*#/)[0].slice(0, -1);
      } 
      console.log(this.apiVerb);   
      this.modifiedTags = eval("this.tags.".concat(this.apiVerb))
    }
  },
  methods: {  
    // select(entry) {
    // //  this.reset(entry);
    // //  this.selectedEntry = entry;
    // //  this.$emit("onSelectEntry", entry);
    // },
    setColor(method_name){
      return method_name+"_color"
    },
    // clickNavigate(summary){
    //  return "#" + summary;
    // }
  }
};



</script>

<style scoped>
.clsEntryMethod {
 width: 66px;
 border-radius: 5px;
 border: 1px dashed #BCB2BB;
}
.clsEntryPath {
 width: calc(100% - 66px);
}
a.clsEntryPath:hover {
  text-decoration:underline;
  }
.v-application .stone-gray-dark {
    background-color: #1d2c36 !important;
    border-color: #1d2c36 !important;
}
.contentOverflow {
    overflow-y: auto;
    height: auto;
    max-height: calc(100vh - 280px);
}
</style>
