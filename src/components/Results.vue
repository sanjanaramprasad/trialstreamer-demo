<template>
<div class="results">
  <div v-show="showExamples">
    <Examples />
  </div>
  <div v-show="!showExamples">
    <transition name="fade" mode="out-in" appear>
      <div v-if="getLoadingArticles"
           key="loading"
           class="d-flex justify-content-center"
           style="margin-top: 4rem; z-index:100; width: 100%">
        <b-img src="@/assets/loading.gif" class="loading-img" style="opacity: 0.2; filter: blur(1px);" height="240" />
          
      </div>
      
      <div v-else key="results" class="result-wrapper">
        <div v-if="sortedArticles.length > 0">
          <p style="font-size: small; text-align: right;">
            first 5 results only
          </p>


          <b-container flex class="p-0">
            <b-row>
                
                
                <b-col cols="auto" class="mr-auto" v-if="getSystemType == 'structured'">
                <b-button-toolbar
                  style="margin-bottom: 2em">
                  <b-button-group>
                     <b-form-radio-group
                      v-model="aspectOpt"
                      :options="aspectOptions"
                      button-variant="light"
                      size="sm"
                      buttons
                      name="radios-btn-default">
                    </b-form-radio-group>
                  </b-button-group>
                </b-button-toolbar>
              </b-col>
                
                
<!--               <b-col cols="auto" class="mr-auto">
                <b-button-toolbar
                  style="margin-bottom: 2em">
                   <b-button-group>
                    <b-form-radio-group
                      v-model="filterType"
                      name="radios-btn-component"
                      button-variant="light"
                      size="sm"
                      buttons>
                      <b-form-radio value="all">All ({{ rowsAll }})</b-form-radio>
                      <b-form-radio value="journal article" :disabled="rowsPublications==0">Published articles ({{ rowsPublications }})</b-form-radio>
                      <b-form-radio value="preprint" :disabled="rowsPreprints==0">Preprints ({{ rowsPreprints }})</b-form-radio>
                      <b-form-radio value="trial registration" :disabled="rowsTrialRegistrations==0">Registered trials ({{ rowsTrialRegistrations}})</b-form-radio>
                    </b-form-radio-group>
                  </b-button-group>
                </b-button-toolbar>
              </b-col> -->
              <b-col cols="auto">
                <b-button-toolbar
                  style="margin-bottom: 2em">
                  <b-button-group>
                    <b-form-radio-group
                      v-model="sortOrder"
                      :options="sortOptions"
                      button-variant="light"
                      size="sm"
                      buttons
                      name="radios-btn-default">
                    </b-form-radio-group>
<!--                     <b-button
                      v-bind:disabled="rows == 0"
                      v-on:click="download"
                      size="sm"
                      v-b-tooltip.hover
                      title="Download citations">
                      <b-icon icon="cloud-download"></b-icon>
                    </b-button> -->
                  </b-button-group>
                </b-button-toolbar>
              </b-col>
            </b-row>
          </b-container>
            
          
          

          <b-container style="margin-bottom: 2em" >
              
          
          
        
          <Summary
            v-bind:aspectOpt="aspectOpt"
            class="result-cards"
            >
          </Summary>
        
          </b-container>

<!--           <Card
            v-for="item in sortedArticles"
            :key="item.pmid"
            v-bind:item="item"
            class="result-cards">
          </Card> -->

          <!-- <div class="d-flex justify-content-center">
            <b-pagination
              v-model="currentPage"
              :total-rows="rows"
              :per-page="perPage"></b-pagination>
          </div> -->
        </div>
        <div v-else class="result-cards" key="no-results">
          <div style="text-align: center; margin-top: 2em">
            <strong>No results</strong>
          </div>
        </div>
      </div>
    </transition>
  </div>
</div>
</template>

<script>
import axios from "axios";
import Examples from "./Examples.vue";
//import Card from "./Card.vue";
import Summary from "./Summary.vue";


function getPaginatedItems(items, page, pageSize) {
  var pg = page || 1,
      pgSize = pageSize || 100,
      offset = (pg - 1) * pgSize,
      pagedItems = window._.drop(items, offset).slice(0, pgSize);
  return pagedItems;
}

export default {
  name: "Results",
  components: { Examples, Summary },
  data() {
    return {
      perPage: 25,
      currentPage: 1,
      sortOrder: 'score',
      aspectOpt: 'model_predicted',
      filterType: 'all',
      sortOptions: [
          {
              text: "Get large/high quality trials first",
              value: 'score',
          },
        {
          text: "Newest first",
          value: 'year',
        },
      ],
      aspectOptions: [
          {
              text: "All aspects",
              value: 'all_aspects',
          },
        {
          text: "Model predicted aspects",
          value: 'model_predicted',
        },
          
      ]
    };
  },
  props: {},
  watch: {
    getTags(to, from) {
      if(!this._.isEqual(to, from)) {
        this.filterType = 'all';
      }
    },
    getArticles(to, from) {
      if(!this._.isEqual(to, from)) {
        this.currentPage = 1;
      }
    }
  },
  methods: {
     
     download: function () {
      axios({
        url: `${process.env.VUE_APP_SERVER_URL}/picosearch`,
        method: "POST",
        data: {
          terms: this.getTags.map((item) => ({
            field: item.classes,
            cui: item.cui,
          })),
          retmode: "ris",
        },
        responseType: "blob",
        headers: {
          "api-key": process.env.VUE_APP_API_KEY,
        },
      }).then((response) => {
        const url = window.URL.createObjectURL(new Blob([response.data]));
        console.log(response.data)
        const link = document.createElement("a");
        link.href = url;
        link.setAttribute("download", "trialstreamer.ris");
        document.body.appendChild(link);
        link.click();
      });
      
    },
  },
  computed: {
    rowsAll() {
      return this.$store.getters.getArticles.length;
    },
    rows() {
      return this.getArticles.length;
    },
    rowsPublications() {
      return this.$store.getters.getArticles.filter(function(el) {return el.article_type=='journal article'}).length;
    },
    rowsPreprints() {
      return this.$store.getters.getArticles.filter(function(el) {return el.article_type=='preprint'}).length;
    },
    rowsTrialRegistrations() {
      return this.$store.getters.getArticles.filter(function(el) {return el.article_type=='trial registration'}).length;
    },
    getLoadingArticles() {
      return this.$store.getters.getLoadingArticles;
    },
    getTags() {
//       alert(JSON.stringify(this.$store.getters.getTags));
      return this.$store.getters.getTags;
    },
    getArticles() {
      let filterType = this.filterType;
      // return this.$store.getters.getArticles.filter(function(el) {return el.article_type==filterType}).length;
      if (this.filterType=='all') {
        return this.$store.getters.getArticles;
      } else {
        return this.$store.getters.getArticles.filter(function(el) {return el.article_type==filterType});
      }
    },
      
    getSystemType(){
        if (process.env.VUE_APP_SUMMARIZE_TYPE.includes('structured')){
                  return 'structured';
                  
              }
        else{
            
            return 'vanilla';
        }
        
    },
      
    isTruncated() {
      return this.getArticles.length >= 250;
    },
    showExamples() {
      return !this.$store.getters.getTags.length && !this.getArticles.length;
    },
    summary() {
      return this.$store.getters.getSummary;
    },
    sortedArticles() {
      let sorttype= this.sortOrder;
      let sortFn = function (a, b) {
        if (sorttype=='score') {
          return b.score - a.score;
        } else {
          return b.year - a.year;
        }
      };
      
      //alert(JSON.stringify(this.$store.getters.getTags));
      let result = this.getArticles.slice().sort(sortFn);
      return getPaginatedItems(result, this.currentPage, this.perPage);
    },
  },
};
</script>

<style scoped>
.results {
  margin-bottom: 2em;
}
.results .results-card:first-child {
  margin-top: 2em;
}
.load-img {
  object-fit: cover;
  width: 100%;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .125s;
}
.fade-enter, .fade-enter-to  {
  opacity: 0;
}

</style>
