<template>
    <b-card>

    
    <h3 > Summary </h3>
    <b-container style="margin-bottom: 2em">
    <!-- <a class="sum" >

    {{this.allArticles[0]}}
    </a> -->
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


<script>

import Card from "./CardSummarySpec.vue";

export default {
  name: "Summary",
  props: ['item', 'allArticles'],
   components: { Card },
  data() {
    return {
      hover: false,
      searchWord: '',
      hover_type: '',
      searchArticlesResults: [],
      color: 'white'
    };
  },

  computed: {
    iterateWords() {
        var sent_words = this.item.punchline_text.split(' ');
        let sent_items = [];
        let colors = ['#003f5c', '#ffa600', '#bc5090', '#007bff']
        var i ;
        for (i = 0; i < sent_words.length; i++) {
            sent_items.push({
                word:sent_words[i],
                color: colors[i%4]
            });
        }
        return sent_items;

    }
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

        }
        

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
