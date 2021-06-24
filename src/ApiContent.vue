<template>
  <div id="goTop" class="px-6 clsOpenSwaggerAPI">
    <hds-btn
            v-scroll="onScroll"
            v-show="fab"
            fab
            dark
            fixed
            bottom
            right
            color="primary"
            @click="toTop"
            title="Go Top"
          >
            <v-icon class="white--text">keyboard_arrow_up</v-icon>
      </hds-btn>

    <v-row>
      <v-col cols="12" sm="12" md="12">
        <h2 class="text-h2-alt stone-gray--text">
          {{ api.info.title }}
        </h2>
        <!-- <div
                    v-if="api.info.description"
                    v-html="marked(api.info.description || '')"
                    style="padding-left: 8px;"
                  ></div> -->
          <div v-for="(selectedEntry, i) in modifiedTags"
            :key="i">
            <div
              class="text-h5 d-inline stone-gray--text"
              v-if="selectedEntry"
            >
            <p v-if="selectedEntry.description">
              <div v-html="selectedEntry.description"></div>
            </p>
            </div>
          </div>
      </v-col>
    </v-row>
    <v-row>
      <v-col cols="12" sm="12" md="12">
        <div class="navActive" :id="selectedEntry.summary" v-for="(selectedEntry, i) in modifiedTags"
        :key="i">
        <div v-if="selectedEntry">
          <h2 class="text-h2-alt fuchsia--text pb-7">
            {{ selectedEntry.title || selectedEntry.summary }}
          </h2>
          <div class="d-flex align-center pb-7">
            <hds-btn
              :id="setColor(selectedEntry.method)"
              class="text-uppercase rounded-pill"
              small
              >{{ selectedEntry.method.toUpperCase() }}</hds-btn
            >
            <h5 class="ml-4 text-h5 d-inline-block stone-gray--text">
              {{ selectedEntry.path }}
            </h5>
          </div>
          <div class="mx-2">
            <p class="entry-description" v-if="selectedEntry.summary" v-html="marked(selectedEntry.summary || '')"></p>
          </div>
          <div v-if="api.servers && api.servers.length" class="mb-3 d-flex">
          

            <!-----Dailog Implementation Starts here----->
            <!-- <hds-dialog v-model="modal" title="Authorize Now" modal>
              <template #title>
                <v-btn icon @click="modal = false">
                  <v-icon>$close</v-icon>
                </v-btn>
              </template>
              <template #text>
                <v-responsive min-height="300" class="my-5">
                  <form @submit.prevent="submit" id="check-auth-form">
                    <h2 class="text-h6 stone-gray--text mb-2">
                      Using Account ID and Secret
                    </h2>
                    <h5 class="text-subtitle-2 stone-gray--text mb-5">
                      Recommended for longer working session, as we will store
                      this information in your session, until you sign out or
                      close the broswer window.
                    </h5>
                    <hds-text-field
                      label="Authorize ID"
                      value="Input text"
                      outlined
                      color="grape"
                      v-model="authID"
                    />
                    <hds-text-field
                      :append-icon="show ? 'mdi-eye' : 'mdi-eye-off'"
                      :type="show ? 'text' : 'password'"
                      label="API Key"
                      v-model="apiKey"
                      @click:append="show = !show"
                      outlined
                      color="grape"
                    ></hds-text-field>
                  </form>
                </v-responsive>
              </template>
              <template #actions>
                <hds-btn alt small @click="modal = false">
                  Close
                </hds-btn>

                <hds-btn
                  class="ml-2"
                  color="primary"
                  min-width="96"
                  small
                  @click="modal = false"
                  type="submit"
                  form="check-auth-form"
                >
                  Authorize
                </hds-btn>
              </template>
            </hds-dialog> -->
            <!---- Dialog Implementation Ends here ---->
            <!-- <hds-btn
              @click.native="emitEntry(selectedEntry)"
              v-bind:href="'#'+ selectedEntry.summary + '_tryItHere'"
              class=""
              color="fuchsia"
              large
            >
              <v-icon class="mr-2 white--text"
                >mdi-hand-pointing-right</v-icon
              >Try It Now</hds-btn
            > -->

            <a href="/files/swagger.json" class="ml-3" download>
              <hds-btn class="text-uppercase" color="grape" large>
                <v-icon class="mr-2 white--text download-animate"
                  >mdi-download</v-icon
                >Try In Postman</hds-btn
              >
            </a>
          </div>
          <v-row class="pt-3">
            <v-col cols="12" sm="12" md="12" class="py-1">
              <div v-if="selectedEntry">
                <hds-card
                  fluid
                  :icon="{
                    icon: 'mdi-table',
                    color: 'stone-gray'
                  }"
                  class="elevation-2"
                  title="Supported Parameters"
                >
                  <template #indented>
                    <parameters-table
                      :selectedEntry="selectedEntry"
                      :openSchemaDialog="openSchemaDialog"
                      :openExamplesDialog="openExamplesDialog"
                    />
                  </template>
                </hds-card>
              </div>
            </v-col>
            <v-col cols="12" sm="12" md="12">
              <div v-if="selectedEntry">
                <div v-if="api.servers && api.servers.length">              
                  <hds-card
                    class="mb-4 elevation-2"
                    :icon="{
                      icon: 'mdi-code-json',
                      color: 'stone-gray'
                    }"
                    title="Body Sample"
                    centered
                    v-if="selectedEntry.method.toUpperCase() !== 'GET' && selectedEntry.method.toUpperCase() !== 'DELETE'"
                  >
                    <template #indented>
                      <requestBody
                        :selectedEntry="selectedEntry"
                        :currentRequest="currentRequest"
                      ></requestBody>
                    </template>

                    <template>
                      <hds-dialog
                        v-model="bodySampleModal"
                        title="Body Sample"
                        modal
                        persistent
                      >
                        <template #activator="{ attrs, on }">
                          <div class="text-center">
                          <hds-btn
                            class="text-h5 white--text"
                            v-bind="attrs"
                            v-on="on"
                            small
                          >
                            <v-icon left class="mr-2 white--text">mdi-arrow-expand-all</v-icon>
                            Expand View
                          </hds-btn>
                          </div>
                        </template>

                        <template #title>
                          <v-btn
                            icon
                            @click="bodySampleModal = false"
                          >
                            <v-icon>$close</v-icon>
                          </v-btn>
                        </template>

                        <template #text>
                          <v-responsive min-height="400">
                            <requestBody
                              :selectedEntry="selectedEntry"
                              :currentRequest="currentRequest"
                            ></requestBody>
                          </v-responsive>
                        </template>

                        <template #actions>
                          <hds-btn
                            alt
                            small
                            color="grape"
                            @click="bodySampleModal = false"
                          >
                            Cancel
                          </hds-btn>
                        </template>
                      </hds-dialog>
                    </template>
                  </hds-card>

                  <!-- <v-card class="borderRadius20 mb-6">
                            <v-card-title class="MSlab_700_24px ml-6 secondary--text">Body Sample</v-card-title>
                            <v-card-text class="ml-2">
                            <requestBody :selectedEntry="selectedEntry" :currentRequest="currentRequest"></requestBody>
                            </v-card-text>
                          </v-card> -->

                  <hds-card
                    class="mb-4 elevation-2"
                    :icon="{
                      icon: 'mdi-code-json',
                      color: 'stone-gray'
                    }"
                    title="Body"                   
                  >
                    <span class="mb-3 clearfix">
                    <hds-btn
                      @click.native="emitEntry(selectedEntry)"
                      v-bind:href="'#'+ selectedEntry.summary + '_tryItHere'"
                      class="text-h5 white--text float-right tryItNowPlacement"
                      color="fuchsia"
                      small
                    >
                      <v-icon class="mr-2 white--text"
                        >mdi-hand-pointing-right</v-icon
                      >Try it Now</hds-btn> 
                    </span> 
                    <template #indented>
                      <div>
                      <request-form
                        ref="requestForm"
                        :selectedEntry="selectedEntry"
                        :currentRequest="currentRequest"
                      ></request-form>
                      </div>
                    </template>
                  </hds-card>

                  <!-- <v-card class="borderRadius20 mb-3">
                            <v-card-title class="MSlab_700_24px ml-6 secondary--text">Body</v-card-title>
                            <v-card-text class="ml-2">
                          <request-form ref="requestForm" :selectedEntry="selectedEntry" :currentRequest="currentRequest"></request-form>
                            </v-card-text>
                          </v-card> -->
                      
                    <div v-if="checkCurrentRequest(selectedEntry)" :id="selectedEntry.summary + '_tryItHere'" class="tryItButtonClick">
                        <hds-card
                          class="mb-4 elevation-2"
                          :icon="{
                            icon: 'mdi-code-json',
                            color: 'stone-gray'
                          }"
                          title="Response Sample"
                          centered
                        >
                          <template #indented>
                            <response-display
                              v-if="currentResponse"
                              :entry="selectedEntry"
                              :response="currentResponse"
                            ></response-display>
                          </template>

                          <template>
                                    <hds-dialog
                                      v-model="modal"
                                      title="Response Sample"
                                      modal
                                      persistent
                                    >
                                      <template #activator="{ attrs, on }">
                                        <div class="text-center">
                                        <hds-btn
                                          class="text-h5 white--text"
                                          v-bind="attrs"
                                          v-on="on"
                                          small
                                        >
                                          <v-icon left class="mr-2 white--text">mdi-arrow-expand-all</v-icon>
                                          Expand View
                                        </hds-btn>
                                        </div>
                                      </template>

                                      <template #title>
                                        <v-btn
                                          icon
                                          @click="modal = false"
                                        >
                                          <v-icon>$close</v-icon>
                                        </v-btn>
                                      </template>

                                      <template #text>
                                        <v-responsive min-height="400">
                                          <response-display
                                            v-if="currentResponse"
                                            :entry="selectedEntry"
                                            :response="currentResponse"
                                          ></response-display>
                                        </v-responsive>
                                      </template>

                                      <template #actions>
                                        <hds-btn
                                          alt
                                          small
                                          color="grape"
                                          @click="modal = false"
                                        >
                                          Cancel
                                        </hds-btn>
                                      </template>
                                    </hds-dialog>
                                  </template>
                        </hds-card>
                    </div>

                    <div v-else :id="selectedEntry.summary + '_tryItHere'" class="tryItButtonClick">
                      <hds-card
                          class="mb-4 elevation-2"
                          :icon="{
                            icon: 'mdi-code-json',
                            color: 'stone-gray'
                          }"
                          title="Response Sample"
                          centered
                        >
                      </hds-card>
                    </div> 

                  <!-- <v-card class="borderRadius20">
                            <v-card-title class="MSlab_700_24px ml-6 secondary--text">Response Sample</v-card-title>
                            <v-card-text class="ml-2">
                              <response-display v-if="currentResponse" :entry="selectedEntry" :response="currentResponse"></response-display>
                            </v-card-text>
                          </v-card> -->
                </div>
              </div>
            </v-col>
          </v-row>
        </div>
        </div>
      </v-col>
    </v-row>
  </div>
</template>

<script>
import Vue from "vue";
import marked from "marked";
import RequestForm from "./RequestForm.vue";
import RequestBody from "./RequestBody.vue";
import ResponseDisplay from "./ResponseDisplay.vue";
import ResponsesTable from "./ResponsesTable.vue";
//import SecurityTable from "./SecurityTable.vue";
import ParametersTable from "./ParametersTable.vue";
import SchemaView from "./SchemaView.vue";
//import Snippet from "./Snippet.vue";
import VueMaterial from "vue-material";
import deref from "json-schema-ref-parser";
import stringify from "json-stringify-pretty-compact";
import VueResource from "vue-resource";
import LeftNavApi from "./LeftNavApi.vue";
import ApiContent from "./ApiContent.vue";

Vue.use(VueMaterial);
Vue.use(VueResource);

export default {
  name: "open-api",
  components: {
    LeftNavApi,
    ApiContent,
    RequestForm,
    RequestBody,
    ResponseDisplay,
    ResponsesTable,
    ParametersTable,
    SchemaView
  },
  props: ["api", "headers", "queryParams", "tags", "selectedEntry"],
  data: () => ({
    drawer: true,
    modifiedTags: null,
    isLoggedIn: localStorage.getItem("jwt") != null,
    apiVerb: null,
    exactVerb: null,
    accordion: false,
    bodySampleModal: false,
    modal: false,
    mini: true,
    authID: "",
    apiKey: "",
    show: false,
    //selectedEntry: null,
    currentSchema: " ",
    currentExamples: {},
    currentFields: {},
    currentRequest: {
      contentType: "",
      body: "",
      params: {},
      security: {}
    },
    currentResponse: null,
    //tags: {},
    httpRequest: null,
    modifiedTags: null,
    apiVerb: "",
    fab: false
  }),
  mounted: function() {
    // if (this.$refs.menu.$children.length)
    //   this.$refs.menu.$children[0].toggleExpandList();
  },
  created() {},
  watch: {
    tags: function() {
      this.apiVerb = window.location.href.split("?")[1];
      if (this.apiVerb.includes("#")){
        this.apiVerb = this.apiVerb.match(/.*#/)[0].slice(0, -1);
        console.log(this.apiVerb)
      } 
      this.exactVerb = this.apiVerb.split("=")[1]
      this.modifiedTags = eval("this.tags.".concat(this.exactVerb));
    },
    selectedEntry: function(){
      if (this.currentResponse) {
        this.currentResponse = null;
      }
      this.request();
    }
  },
  methods: {
    marked,
    checkCurrentRequest(selectedEntry){
      if (this.selectedEntry) {
        return (selectedEntry.path == this.selectedEntry.path && selectedEntry.method == this.selectedEntry.method)
      }
    },
    scrollToTop() {
        window.scrollTo(0,0);
    },
    onScroll (e) {
      if (typeof window === 'undefined') return
      const top = window.pageYOffset ||   e.target.scrollTop || 0
      this.fab = top > 20
    },
    toTop () {
      this.$vuetify.goTo(0)
    },
    stringify,
    // reset(entry) {
    //   const newParams = {};
    //   (entry.parameters || []).forEach((p) => {
    //     newParams[p.name] =
    //       (p.in === "query" && this.queryParams && this.queryParams[p.name]) ||
    //       (p.in === "header" && this.headers && this.headers[p.name]) ||
    //       null;
    //     if (!newParams[p.name]) {
    //       if (p.schema && p.schema.enum) {
    //         newParams[p.name] = p.schema.enum[0];
    //       }
    //       if (p.schema && p.schema.type === "array") {
    //         newParams[p.name] = [];
    //       }
    //       if (p.example) {
    //         newParams[p.name] = p.example;
    //       }
    //     }
    //   });
    //   this.currentRequest.params = newParams;

    //   const newSecurity = {};
    //   entry.security.forEach((s) => {
    //     this.$set(
    //       newSecurity,
    //       s.scheme.name,
    //       this.currentRequest.security[s.scheme.name] || null
    //     );
    //   });
    //   this.currentRequest.security = newSecurity;

      // if (entry.requestBody) {
      //   this.currentRequest.contentType = entry.requestBody.selectedType;
      //   const example =
      //     entry.requestBody.content[this.currentRequest.contentType].example;
      //   this.currentRequest.body =
      //     typeof example === "string" ? example : stringify(example, null, 2);
      // }
    // },
    // select(entry) {
    //   this.reset(entry);
    //   this.selectedEntry = entry;
    // },
    setColor(method_name) {
      return method_name + "_Bgcolor";
    },
    openSchemaDialog(schema) {
      alert("open schema dialog triggered");
      this.currentSchema = schema;
      this.$refs.schemaDialog.open();
    },
    openExamplesDialog(examples) {
      this.currentExamples = examples;
      this.$refs.examplesDialog.open();
    },
    openFieldsDialog(fields) {
      this.currentFields = fields;
      this.$refs.fieldsDialog.open();
    },
    prepareHTTPRequest() {
      const entry = this.selectedEntry;
      const request = this.currentRequest;
      if (!entry || !request) return;
      let params = Object.assign(
        {},
        ...(entry.parameters || [])
          .filter(
            p =>
              p.in === "query" &&
              (p.schema.type === "array"
                ? request.params[p.name].length
                : request.params[p.name])
          )
          .map(p => ({
            // TODO : join character for array should depend of p.style
            [p.name]:
              p.schema.type === "array"
                ? request.params[p.name].join(",")
                : request.params[p.name]
          }))
      );
      let headers = Object.assign(
        {},
        ...(entry.parameters || [])
          .filter(
            p =>
              p.in === "header" &&
              (p.schema.type === "array"
                ? request.params[p.name].length
                : request.params[p.name])
          )
          .map(p => ({
            // TODO : join character for array should depend of p.style
            [p.name]:
              p.schema.type === "array"
                ? request.params[p.name].join(",")
                : request.params[p.name]
          }))
      );

      entry.security
        .filter(s => !!request.security[s.scheme.name])
        .forEach(s => {
          if (s.scheme.in === "header") {
            headers[s.scheme.name] = request.security[s.scheme.name];
          } else if (s.scheme.in === "query") {
            params[s.scheme.name] = request.security[s.scheme.name];
          }
        });

      const httpRequest = {
        method: entry.method,
        url:
          this.api.servers.length &&
          this.api.servers[0].url +
            entry.path.replace(/{(\w*)}/g, (m, key) => {
              return request.params[key];
            }),
        params,
        headers,
        credentials: true
      };

      const contentType =
        this.selectedEntry.requestBody &&
        this.selectedEntry.requestBody.selectedType;
      if (contentType) {
        headers["content-type"] = this.selectedEntry.requestBody.selectedType;
        if (contentType === "multipart/form-data") {
          httpRequest.body = this.$refs.requestForm.getFormData();
        } else {
          httpRequest.body = request.body;
        }
      }

      this.httpRequest = httpRequest;
    },
    emitEntry(entry){     
      if (this.selectedEntry) {
        if (entry.path == this.selectedEntry.path && entry.method == this.selectedEntry.method) {
          this.request();
        } else {
          this.$emit("onSelectEntry", entry);
        }
      } else {
        this.$emit("onSelectEntry", entry);
      }
    },
    request() {
      if (!this.isLoggedIn) {
        //this.modal = true;
      } else {
        document.getElementById("overlay").style.display = "block";
        this.prepareHTTPRequest();
        Vue.http(this.httpRequest).then(
          res => {
            this.currentResponse = res;
             document.getElementById("overlay").style.display = "none";
          },
          res => {
            this.currentResponse = res;
            document.getElementById("overlay").style.display = "none";
          }
        );
      }
    }
  }
};

/*
 * Tags management utils
 */

const defaultStyle = {
  query: "form",
  path: "simple",
  header: "simple",
  cookie: "form"
};

function processContent(contentType, api) {
  // Spec allow examples as an item or an array. In the API or in the schema
  // we always fall back on an array
  if (contentType.schema) {
    contentType.examples = contentType.examples || contentType.schema.examples;
    contentType.example = contentType.example || contentType.schema.example;
  }

  if (contentType.example) {
    contentType.examples = [contentType.example];
  }
}

async function getTags(api) {
  const derefAPI = await deref.dereference(api);
  const tags = {};
  Object.keys(derefAPI.paths).forEach(path => {
    Object.keys(derefAPI.paths[path])
      .filter(
        method =>
          [
            "get",
            "put",
            "post",
            "delete",
            "options",
            "head",
            "patch",
            "trace"
          ].indexOf(method.toLowerCase()) !== -1
      )
      .forEach(method => {
        const entry = derefAPI.paths[path][method];
        entry.method = method;
        entry.path = path;
        // Filling tags entries
        entry.tags = entry.tags || [];
        if (!entry.tags.length) {
          entry.tags.push("No category");
        }
        entry.tags.forEach(tag => {
          tags[tag] = tags[tag] || [];
          tags[tag].push(entry);
        });

        // Add path level parameters to the operation
        entry.parameters = entry.parameters || [];
        if (derefAPI.paths[path].parameters) {
          entry.parameters = derefAPI.paths[path].parameters.concat(
            entry.parameters
          );
        }
        if (entry.parameters) {
          entry.parameters.forEach(p => {
            p.style = p.style || defaultStyle[p.in];
            p.explode = p.explode || p.style === "form";
            p.schema = p.schema || { type: "string" };
          });
        }
        if (entry.requestBody) {
          if (
            entry.requestBody.content &&
            Object.keys(entry.requestBody.content).length
          ) {
            Vue.set(
              entry.requestBody,
              "selectedType",
              Object.keys(entry.requestBody.content)[0]
            );
            entry.requestBody.required = true;
            Object.values(entry.requestBody.content).forEach(contentType =>
              processContent(contentType, api)
            );
          }
        }

        // security comes from the root ofthe API or the operation
        entry.security = entry.security || derefAPI.security || [];
        entry.security.forEach(s => {
          const key = Object.keys(s)[0];
          s.key = key;
          s.scheme = derefAPI.components.securitySchemes[key];
        });

        // Some preprocessing with responses
        entry.responses = entry.responses || {};
        Object.values(entry.responses).forEach(response => {
          if (response.content) {
            // preselecting responses mime-type
            Vue.set(response, "selectedType", Object.keys(response.content)[0]);
            Object.values(response.content).forEach(contentType =>
              processContent(contentType, api)
            );
          }
        });
      });
  });
  return tags;
}
</script>

<style scoped>
/* .v-application .theme--light >>> .text-h2 {
    color: #444fa2 !important
} */
.clsOpenSwaggerAPI .text-h5 {
  line-height: 1.5rem !important;
}
.clsOpenSwaggerAPI .v-icon {
  height: auto;
  width: auto;
}
.download-animate {
  -webkit-animation: animate 3s infinite;
  animation: animate 3s infinite;
}
.v-icon.download-animate:hover {
  animation: none;
}
@keyframes animate {
  0% {
    transform: translateY(-3px);
  }
  100% {
    transform: translateY(3px);
    opacity: 0.2;
  }
}
.navActive, .tryItButtonClick{
  padding-top: 95px;
  margin-top: -95px;
  scroll-behavior: smooth;
}
.tryItNowPlacement{
  position:absolute;
  top:28px;
  right:22px;
}
</style>
