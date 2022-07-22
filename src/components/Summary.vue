<template>
    <b-card>
      
    <b-container style="margin-bottom: 4em" v-if="this.vanilla_summary != '' && this.system == 'structured'">
        
        <div style="background: #dedfe0; padding: 0.5rem; border-radius: 0.2rem"> 
            <h6> Template </h6>
            <select v-on:input="getSummary($event.target.value)"  > 
                <option value="vanilla" key="vanilla"> General Summary </option>
                <option value="template_diff" key="template_diff"> __interventions__ appears to have a beneficial effect on __outcomes__ in patients with __population__</option>
                <option value="template_nodiff" key="template_nodiff">There is no  evidence that __interventions__ has an effect on __outcomes__ in patients with __population__  </option>
            </select>
        </div>
    </b-container>
        
    <div v-if="this.summary == ''"
           key="loading"
           class="d-flex justify-content-center"
           style="margin-top: 4rem; z-index:100; width: 100%">
        <b-img src="@/assets/loading.gif" class="loading-img" style="opacity: 0.2; filter: blur(1px);" height="240" />
        
      </div>
    
    <b-container style="margin-bottom: 2em">
        
    <div v-if="this.summary != ''">
    <h3 > Summary </h3>
    </div>
    <div class="sum" v-for=" i in iterateWords" :key="i" >
        <span
            style="text-decoration: underline var(--population-background)"
            v-if="i.color == '#003f5c'"
            @click="mouseIn(i.word, 'pop')"
            v-bind:class='{ "population_token_clicked": (hover_type == "pop" || hover_type == "all") && (searchWord == i.word), "population_token" : hover_type != "pop" || searchWord != i.word}'>
                {{i.word}}
        </span>

        <span 
            style="text-decoration: underline var(--intervention-background)"
            v-if="i.color == '#ffa600'"
            class ="intervention_token"
            @click="mouseIn(i.word, 'int')"
            v-bind:class='{ "intervention_token_clicked": (hover_type == "int" || hover_type == "all") && (searchWord == i.word), "intervention_token" : hover_type != "int" || searchWord != i.word}'>
                {{i.word}}
        </span>

        <span 
            style="text-decoration: underline var(--outcome-background)"
            v-if="i.color == '#bc5090'"
            class ="outcome_token"
            @click="mouseIn(i.word, 'out')"
            v-bind:class='{ "outcome_token_clicked": (hover_type == "out" || hover_type == "all") && (searchWord == i.word), "outcome_token" : hover_type != "out" || searchWord != i.word}'>
                {{i.word}}
        </span>

        <span
            style="text-decoration: underline var(--gray)"
            v-if="i.color == '#007bff'"
            class ="ptext_token"
            @click="mouseIn(i.word, 'ptext')"
            v-bind:class='{ "ptext_token_clicked": (hover_type == "ptext" || hover_type == "all") && (searchWord == i.word), "ptext_token" : hover_type != "ptext" || searchWord != i.word}'>
                {{i.word}}
        </span>
        
        <span
            v-if="i.color == 'white' "
            class ="other_token">
                {{i.word}}
        </span>

    
    </div>
    
    </b-container>
    
    <b-container style="margin-bottom: 2em">
        
    <div v-if="this.summary != ''">
        <!--         {{getData()}} -->
         <button @click="copy">Click to copy for Annotation</button>
    </div>
        
    <div v-if="(this.summary != '') && ((hover) || (iterateWords && this.system == 'vanilla'))" > 
    <!-- <Card
            v-for="item in this.searchArticlesResults"
            :key="item.pmid"
            v-bind:item="item"
            v-bind:hover_type="hover_type"
            v-bind:searchWord="searchWord"
            class="result-cards">
          </Card> -->
    
    <Card
            v-bind:searchArticlesResults="searchArticlesResults"
            v-bind:otherArticlesResults="otherArticlesResults"
            v-bind:hover_type="hover_type"
            v-bind:searchWord="searchWord"
            class="result-cards">
          </Card>

    </div>
    </b-container>
    </b-card>

    
    <!-- <a class="sum"  v-for=" i in iterateWords" :key="i" 
        v-bind:style="{ 'background-color': i.color, 'color': white }"> 
        
        {{i.word}} 
    </a> -->
    
    
    

  
</template>
<!-- {'summary': 'ZS  may  be safe  and', 'aspect_indices': ['population', 'interventions', 'interventions', 'other', 'outcomes', 'outcomes']} -->

<script>

import Card from "./CardSummarySpec.vue";
import axios from "axios";
export default {
  name: "Summary",
  props: [ 'allArticles', 'aspectOpt'],
   components: { Card },
  data() {
    return {
      hover: false,
      searchWord: '',
      hover_type: '',
      summaryType: 'vanilla',
      searchArticlesResults: [],
      otherArticlesResults: [],
      color: 'white',
      summary: '',
      aspects: [],
      vanilla_summary: '',
      vanilla_aspects: [],
      template_diff_summary:'',
      template_diff_aspects:[],
      template_nodiff_summary: '',
      template_nodiff_aspects: [],
      autocompleteItems: [],
      clipboardData: {},
      system: '',
      has_aspects: true,
      labels: [],
      
    };
  },

//   vanilla_summary(){
      
      
//   },

  mounted() {
            //const url = `http://localhost:5001/${process.env.VUE_SUMMARIZE_TYPE}`
            //const path = 'http://localhost:5001/summarize'
                axios({
                      url: `${process.env.VUE_APP_SUMM_URL}/${process.env.VUE_APP_SUMMARIZE_TYPE}`,
                      method: 'POST',
                      data : this.allArticles.slice(0,5)
                    })
                    .then(response => {
                      this.summary = response.data['summary'];
                      this.vanilla_summary = response.data['summary'];
                      this.aspects = response.data['aspect_indices'];
                      this.vanilla_aspects = response.data['aspect_indices'];
                      if (process.env.VUE_APP_SUMMARIZE_TYPE.includes('structured')){
                          this.system = 'structured';
                          this.has_aspect = true;
                          

                      }
                      else{

                          this.system = 'vanilla';
                          this.has_aspect = false;
                          this.hover_type = 'all';
                          this.searchArticlesResults = this.allArticles.slice(0,5);
                      }

                    })
                    .catch(err =>{
                      alert(JSON.stringify(err));
                    });

                
      
              
            
            },
    
  computed: {
    
      
    
        iterateWords() {
            var sent_words = this.summary.split(' ');
            let sent_items = [];
            var colors_map = {'population': '#003f5c', 
                              'interventions': '#ffa600', 
                              'outcomes': '#bc5090', 
                              'punchline_text':'#007bff',
                              'other': 'white'};
            var i ;
            
            for (i = 0; i < sent_words.length; i++) {
                
                if (process.env.VUE_APP_SUMMARIZE_TYPE.includes('structured')){
                    sent_items.push({
                        word:sent_words[i],
                        color: colors_map[this.aspects[i]]
                    });
                }
                else{
                    
                    sent_items.push({
                        word:sent_words[i],
                        color: colors_map['other']});
                }
            }
        
            
            return sent_items;

        },
      
        
      
        
   },

methods: {
    
    
        directReflect(summ_data){
            axios({
                      url: `${process.env.VUE_APP_SUMM_URL}/reflect`,
                      method: 'POST',
                      data : summ_data
                    })
                    .then(response => {
                      this.summary = response.data['summary'];
                      this.aspects = response.data['aspect_indices'];

                    })
                    .catch(err =>{
                      alert(JSON.stringify(err));
                    });
            
        },
    
        directSummarizeTemplate(summ_data){
            axios({
                    url: `${process.env.VUE_APP_SUMM_URL}/template`,
                    method: 'POST',
                    data : summ_data
            })
            .then(response => {
                this.summary = response.data['summary'];
                this.aspects = response.data['aspect_indices'];
                if (summ_data['direc'] == 'diff'){
                    this.template_diff_summary = response.data['summary'];
                    this.template_diff_aspects = response.data['aspect_indices'];
                }
                else{
                    this.template_nodiff_summary = response.data['summary'];
                    this.template_nodiff_aspects = response.data['aspect_indices'];
                }
            })
            .catch({ 
              });
            
        },
    
        getSummary(summaryType){
            this.summary = '';
            if(summaryType == 'vanilla'){
                    this.directReflect({'aspect_indices' : this.vanilla_aspects, 'summary': this.vanilla_summary});
                  
              }
            
            else if(summaryType == 'template_diff'){
                  if (this.template_diff_summary != '' && (typeof this.template_diff_summary !== 'undefined')){
                        this.directReflect({'aspect_indices' : this.template_diff_aspects, 'summary': this.template_diff_summary});

                      }
             
                
                  else { 
                         this.directSummarizeTemplate({'data':this.allArticles.slice(0,5), 'summary': this.vanilla_summary, 'direc': 'diff'});
                      
                        
                  }
                  
            }
            
            else if(summaryType == 'template_nodiff'){
                  if (this.template_nodiff_summary != '' && (typeof this.template_nodiff_summary !== 'undefined')){
                        this.directReflect({'aspect_indices' : this.template_nodiff_aspects, 'summary': this.template_nodiff_summary});

                      }
             
                
                  else { 
                         this.directSummarizeTemplate({'data':this.allArticles.slice(0,5), 'summary': this.vanilla_summary, 
                                                       'direc':'nodiff'});
                      

                  }
                  
            }
            
            
            
        },
    
        getData() {
//           alert(JSON.stringify(this.$store.getters.getTags));
          this.clipboardData['system'] = this.system;
          this.clipboardData['has aspects'] = this.has_aspect;
          this.clipboardData['summary'] = this.summary.split(' ');
          this.clipboardData['search terms'] = this.$store.getters.getTags;
          let terms = {};
          var i;
          var searches = this.$store.getters.getTags;
          for (i = 0; i < searches.length; i++){
              var search_item = searches[i];
              var aspect;
//               [{"field":"population","text":"COVID-19 [population]","cui":"TS-COV19"},{"field":"interventions","text":"Remdesivir [interventions]","cui":"C4726677"}]
              var text = search_item["text"];
              
              if(text.includes('population')){
                  aspect = 'population';
              }
              
              else if(text.includes('interventions')){
                  aspect = 'interventions'
                  
              }
              else if(text.includes('outcomes')){
                  
                  aspect = 'outcomes'
              }
              
//               alert(aspect);
//               [{"field":"population","text":"COVID-19 [population]","cui":"TS-COV19"},{"field":"interventions","text":"Remdesivir [interventions]","cui":"C4726677"}]
//               var aspect = search_item["classes"];
//               var text = search_item["text"];
              
              var name_map = {'population': 'population', 
                              'interventions': 'intervention', 
                              'outcomes': 'outcome'};
              terms[name_map[aspect]] =  text;
              
              
            
              
          }
          this.clipboardData['search terms'] = terms;
          this.clipboardData['studies'] = []
          for(var j = 0; j < this.allArticles.slice(0,1).length; j++){
              var study_dict = {};
              var article_j = this.allArticles[j];
              study_dict['title'] = article_j['ti'];
              study_dict['punchline'] = article_j['punchline_text'];
              study_dict['populations'] = article_j['population'];
              study_dict['interventions'] = article_j['interventions'];
              study_dict['outcomes'] = article_j['outcomes'];
              this.clipboardData['studies'].push(study_dict);
          }
            
        
          if (this.has_aspect){
              this.clipboardData['label names'] = this.aspects;
              this.clipboardData['labels'] = []
              var aspect_map = {'population': 0, 
                              'interventions': 1, 
                              'outcomes': 2, 
                              'punchline_text':3,
                               'other': 4};
              for (i = 0; i < this.aspects.length; i++){
                  var a = this.aspects[i];
                  this.clipboardData['labels'].push(aspect_map[a]);
                  
              }
              
          }
         
          return this.clipboardData;
        },
    
        copy() {
            this.getData();
            var copyText = JSON.stringify(this.clipboardData);
            //alert(copyText);
            var el = document.createElement('textarea');
            el.value = copyText;
            el.setAttribute('readonly', '');
            el.style = {
                position: 'absolute',
                left: '-9999px'
            };
            document.body.appendChild(el);
            el.select();
            document.execCommand('copy');
            document.body.removeChild(el);
        },
    
        mouseIn(searchWord, hover_type) {
            this.searchArticlesResults = [];
            this.otherArticlesResults = [];
            this.hover = false;
            
            if(this.aspectOpt == 'all_aspects' || hover_type == 'other'){
                this.hover_type ='all';
            }
            else{
                this.hover_type = hover_type;
            }
            
            this.searchWord = searchWord;
            var candidates;
            //alert(this.hover_type);
            for(var j = 0; j < this.allArticles.slice(0,5).length; j++){

                candidates = [];
                if (this.hover_type == 'pop' || this.hover_type == 'all'){
                    candidates = this.allArticles[j].population;
                }

                else if (this.hover_type == 'int' || this.hover_type == 'all'){
                    candidates = this.allArticles[j].interventions;
                    
                }
                else if (this.hover_type == 'out' || this.hover_type == 'all'){
                    candidates = this.allArticles[j].outcomes;
                }

                else if (this.hover_type == 'ptext' || this.hover_type == 'all'){
                    candidates = [this.allArticles[j].punchline_text];
                    
                }

                var append_flag = false;
                for(var k = 0; k < candidates.length; k++){
                    let result = candidates[k];
                    
                    if(result.includes(this.searchWord.trim())){
                        append_flag = true;
                    }
                }

                if (append_flag){
                    this.searchArticlesResults.push(this.allArticles[j]);
                }
                else{
                   this.otherArticlesResults.push(this.allArticles[j]);
                    
                }
                
                
                    
            }
            //alert('ended');
            this.hover = true;
            
            
            
        },

        changeColor(word, hover_type){
            if(hover_type == 'pop' && word == this.searchWord){
                this.color = "red";
            }
            else{
                this.color = 'white';
            }
            
        },

        mouseOut() {
            this.hover = false;

        },
    
    
         // summarizeData: function () {
         //        const path = 'http://localhost:5001/summarize'
         //    axios.post(path, {
         //      "data" : this.allArticles
         //    })
         //    .then(response => {
         //      this.summary = response.data['summary'];
         //      this.aspects = response.data['aspect_indices'];
         //      // alert(JSON.stringify(response.data));
         //    })
         //    .catch(err =>{
         //      alert(JSON.stringify(err));
         //    });
         //    },
        
        
        

}

  
}
</script>

<style scoped lang="scss">

.population_token{
    background-color: white;
    color:black;
    display: inline;
}

.population_token:hover{
    background-color: var(--population-background);
    box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15) !important;
    display: inline;
    color:white;
}

.intervention_token{
    background-color: white;
    color:black;
    display: inline;
}

.intervention_token:hover {
    background-color: var(--intervention-background);
    box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15) !important;
    display: inline;
    color:white;
}


.outcome_token{
    background-color: white;
    color:black;
    display: inline;
}

.outcome_token:hover{
    background-color: var(--outcome-background);
    box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15) !important;
    display: inline;
    color:white;
}

.ptext_token{
    background-color: white;
    color:black;
    display: inline;
}
.other_token{
    background-color: white;
    color:black;
    display: inline;
}

.ptext_token:hover{
    background-color: #6f7070;
    box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15) !important;
    display: inline;
    color:white;
}
.population_token_clicked {
    background-color: var(--population-background);
    display: inline;
    color:white;
}

.intervention_token_clicked {
    background-color: var(--intervention-background);
    display: inline;
    color:white;
}

.outcome_token_clicked {
    background-color: var(--outcome-background);
    display: inline;
    color:white;
}

.ptext_token_clicked {
    background-color: #6f7070;
    display: inline;
    color:white;
}
.sum{
    display:inline;
}


</style>
