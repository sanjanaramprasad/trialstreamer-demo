<template>
    <b-card>

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
            v-if="i.color == '#003f5c'"
            @click="mouseIn(i.word, 'pop')"
            v-bind:class='{ "population_token_clicked": hover_type == "pop" && searchWord == i.word, "population_token" : hover_type != "pop" || searchWord != i.word}'>
                {{i.word}}
        </span>

        <span 
            v-if="i.color == '#ffa600'"
            class ="intervention_token"
            @click="mouseIn(i.word, 'int')">
                {{i.word}}
        </span>

        <span 
            v-if="i.color == '#bc5090'"
            class ="output_token"
            @click="mouseIn(i.word, 'out')">
                {{i.word}}
        </span>

        <span
            v-if="i.color == '#007bff'"
            class ="ptext_token"
            @click="mouseIn(i.word, 'ptext')">
                {{i.word}}
        </span>
        
        <span
            v-if="i.color == 'white'"
            class ="other_token">
                {{i.word}}
        </span>

    
    </div>
    </b-container>
    
    <b-container style="margin-bottom: 2em">
    <div v-if="hover" > 
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
  props: [ 'allArticles'],
   components: { Card },
  data() {
    return {
      hover: false,
      searchWord: '',
      hover_type: '',
      searchArticlesResults: [],
      color: 'white',
      summary: '',
      aspects: [],
      autocompleteItems: [],
    };
  },

  mounted() {
                const path = 'http://localhost:5001/summarize'
            axios.post(path, {
              "data" : this.allArticles
            })
            .then(response => {
              this.summary = response.data['summary'];
              this.aspects = response.data['aspect_indices'];
              // alert(JSON.stringify(response.data));
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
                sent_items.push({
                    word:sent_words[i],
                    color: colors_map[this.aspects[i]]
                });
            }
            alert(JSON.stringify(sent_items));
            return sent_items;

        },
   },

methods: {
        mouseIn(searchWord, hover_type) {
            this.searchArticlesResults = [];
            this.hover = true;
            this.hover_type = hover_type;
            this.searchWord = searchWord;
            var candidates;

            for(var j = 0; j < this.allArticles.length; j++){


                if (this.hover_type == 'pop'){
                    candidates = this.allArticles[j].population;
                }

                else if (this.hover_type == 'int'){
                    candidates = this.allArticles[j].interventions;
                }
                else if (this.hover_type == 'out'){
                    candidates = this.allArticles[j].outcomes;
                }

                else if (this.hover_type == 'ptext'){
                    candidates = [this.allArticles[j].punchline_text];
                    
                }

                var append_flag = false;
                for(var k = 0; k < candidates.length; k++){
                    let result = candidates[k];
                    
                    if(result.includes(this.searchWord)){
                        //alert(result.includes(this.searchWord));
                        append_flag = true;
                    }
                }

                if (append_flag){
                    this.searchArticlesResults.push(this.allArticles[j])
                }
                
                
                    
            }
            

            
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
            //this.hover = false;

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


.output_token{
    background-color: white;
    color:black;
    display: inline;
}

.output_token:hover{
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
