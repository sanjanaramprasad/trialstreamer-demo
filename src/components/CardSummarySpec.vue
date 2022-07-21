<template>


<div class="result-card shadow-sm p-3 mb-5 rounded"  >
    
    
<!--     {{this.hover_type}} -->
    <h6>
        Overview
    </h6>
    
    <div class="row">
        <div class="card" v-for="(item, index) in this.searchArticlesResults" :key="item">

            <blockquote >
                    <h6> Study {{index + 1}} </h6>
                    <h6 >{{ item.ti }}</h6>
            </blockquote>


            <h6 v-if="item.article_type == 'preprint' ||  item.article_type == 'journal article'">
                        <a
                          v-bind:href="`https://www.ncbi.nlm.nih.gov/pubmed/${item.pmid}`"
                          target="_blank">{{ item.pmid }}</a>
                        {{ item.citation }}
                        <a
                          v-if="item.dois && item.dois[0]"
                          v-bind:href="`https://dx.doi.org/${item.dois[0]}`"
                          target="_blank">{{ item.dois[0] }}</a>
            </h6>


            <span v-html="highlight_pop(item.punchline_text , 'ptext')"> {{item.punchline_text}} </span>      

        </div>
        
        <div class="card" v-for="(item, index) in this.otherArticlesResults" :key="item">

            <blockquote >
                    <h6> Study {{ getIndex(index) }} </h6>
                    <h6 >{{ item.ti }}</h6>
            </blockquote>


            <h6 v-if="item.article_type == 'preprint' ||  item.article_type == 'journal article'">
                        <a
                          v-bind:href="`https://www.ncbi.nlm.nih.gov/pubmed/${item.pmid}`"
                          target="_blank">{{ item.pmid }}</a>
                        {{ item.citation }}
                        <a
                          v-if="item.dois && item.dois[0]"
                          v-bind:href="`https://dx.doi.org/${item.dois[0]}`"
                          target="_blank">{{ item.dois[0] }}</a>
            </h6>


            <span> {{item.punchline_text}} </span>      

        </div>
        
    </div>
    

    
<!-- Population cards begin -->
<!--     <span style="text-decoration: solid underline var(--population-background) 4px;" v-if="(hover_type == 'pop' || hover_type == 'all')">
        Population
    </span> -->
    <div class="row" v-if="(hover_type == 'pop' || hover_type == 'all')">
        
        
        <div class="card" style="border-color: var(--population-background); border-width: thick;" v-for="(item, index) in this.searchArticlesResults" :key="item">    
            
            <ul>
                    <h6> Study {{index + 1}} </h6>
                    <li v-for="p in distinct(item.population)" :key="p">
                             <span >
                              {{ p }}
                             </span>
                    </li>
                    <li v-if="!item.population.length">
                            <em>None extracted</em>
                    </li>
            </ul>       

        </div>
        
        <div class="card" style="border-color: var(--population-background); border-width: thick;" v-for="(item, index)  in this.otherArticlesResults" :key="item">



            <ul>
                    <h6> Study {{ getIndex(index) }} </h6>
                    <li v-for="p in distinct(item.population)" :key="p">
                             <span >
                              {{p }}
                             </span>
                    </li>
                    <li v-if="!item.population.length">
                            <em>None extracted</em>
                    </li>
            </ul>       

        </div>
    </div>
<!-- Population cards end -->

<!-- Interventions cards begin -->
    <span style="text-decoration:solid underline var(--intervention-background) 4px;" v-if="(hover_type == 'int' || hover_type == 'all')">
        Interventions
    </span>
    <div class="row" v-if="(hover_type == 'int' || hover_type == 'all')">
        
        <div class="card" style="border-color: var(--intervention-background); border-width: thick;" v-for="(item, index) in this.searchArticlesResults" :key="item">    

            <ul>
                    <h6> Study {{index + 1}} </h6>
                    <li v-for="i in distinct(item.interventions)" :key="i">
                             <span >
                              {{fixParens(i) }}
                             </span>
                    </li>
                    <li v-if="!item.interventions.length">
                            <em>None extracted</em>
                    </li>
            </ul>       

        </div>
        
        <div class="card" style="border-color: var(--intervention-background); border-width: thick;" v-for="(item, index) in this.otherArticlesResults" :key="item">



            <ul>
                    <h6> Study {{ getIndex(index) }} </h6>
                    <li v-for="i in distinct(item.interventions)" :key="i">
                             <span>
                              {{fixParens(i) }}
                             </span>
                    </li>
                    <li v-if="!item.interventions.length">
                            <em>None extracted</em>
                    </li>
            </ul>       

        </div>
    </div>
<!-- Interventions cards end -->
    
<!-- Outcomes cards begin -->
    <span style="text-decoration: solid underline var(--outcome-background) 4px;" v-if="(hover_type == 'out' || hover_type == 'all')">
        Outcomes
    </span>
    <div class="row" v-if="(hover_type == 'out' || hover_type == 'all')">
        
        <div class="card" style="border-color: var(--outcome-background); border-width: thick;" v-for="(item, index) in this.searchArticlesResults" :key="item">    

            <ul>
                    <h6> Study {{index + 1}} </h6>
                    <li v-for="o in distinct(item.outcomes)" :key="o">
                             <span >
                              {{fixParens(o) }}
                             </span>
                    </li>
                    <li v-if="!item.outcomes.length">
                            <em>None extracted</em>
                    </li>
            </ul>       

        </div>
        
        <div class="card" style="border-color: var(--outcome-background); border-width: thick;" v-for="(item, index) in this.otherArticlesResults" :key="item">



            <ul>
                    <h6> Study {{ getIndex(index) }} </h6>
                    <li v-for="o in distinct(item.outcomes)" :key="o">
                             <span >
                              {{fixParens(o) }}
                             </span>
                    </li>
                    <li v-if="!item.outcomes.length">
                            <em>None extracted</em>
                    </li>
            </ul>       

        </div>
    </div>
<!-- Outcomes cards end -->
    
    
</div> 
</template>


<script>
export default {
  name: "Card",
  props: ['searchArticlesResults', 'otherArticlesResults','hover_type', 'searchWord'],
  computed: {
    
  },
  methods: {
    // https://stackoverflow.com/questions/26586753/javascript-add-missing-parentheses-in-string
    _fixParens: function(s, left, right) {
      var missedOpen = 0, missedClosed = 0;
      for (let i = 0; i < s.length; i++) {
        if (s[i] == left) {
          missedClosed++;
        } else if (s[i] == right) {
          if (missedClosed > 0)
            missedClosed--;
          else
            missedOpen++;
        }
      }
      return Array(missedOpen + 1).join(left) + s + Array(missedClosed + 1).join(right);
    },
    stripHtml: function(html)
    {
      var tmp = document.createElement("DIV");
      tmp.innerHTML = html;
      return tmp.textContent || tmp.innerText || "";
    },
    fixParens: function(s) {
      s = this.stripHtml(s)
      let a = this._fixParens(s,"(", ")");
      let b = this._fixParens(a,"[","]");
      return b;
    },
    getIndex: function(i) {
      return i + this.searchArticlesResults.length + 1
    },
    highlight_pop(s, s_type) {
//         alert(s);
        if(this.searchWord === '') { return s }
        
        // when removing the .toLowerCase() your search becomes case-sensitive
        if (s_type == 'pop'){
          return s.replace(this.searchWord, '<span style="background: yellow; color: black;">' + this.searchWord + '</span>')
        }
        else if (s_type == 'int'){

          return s.replace(this.searchWord, '<span style="background: yellow; color: black;">' + this.searchWord + '</span>')
        }
        else if (s_type == 'out'){
          
          return s.replace(this.searchWord, '<span style="background: var(--outcome-background); color: white;">' + this.searchWord + '</span>')
        }
        else if (s_type == 'ptext'){

          return s.replace(this.searchWord, '<span style="background: #6f7070; color: white;">' + this.searchWord + '</span>')
        }
       
    },
    distinct: function (strs) {
      let d = new Map(strs.map((s) => [s.toLowerCase(), s]));
      return [...d.values()].filter(n => n);
    }
  }
}
</script>

<style scoped lang="scss">
.result-card {
  margin-top: 0;
  text-align: left;
  &:hover {
    .article-type {
      color: white;
    }
    .article-type[data-article-type="journal article"] {
      background-color: var(--blue);
    }
    .article-type[data-article-type="preprint"] {
      background-color: var(--indigo);
    }
    .article-type[data-article-type="trial registration"] {
      background-color: var(--cyan);
    }
  }
}
.num-randomized {
  cursor: pointer;
}
.result-card,
.result-card * {
  transition: all 0.125s ease !important;
}
.result-card .pico-col:last-child {
  margin-bottom: 2em;
}

.result-card .dim-badge {
  transition: initial;
}
.result-card:hover {
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15) !important;
}
.result-card:not(:hover) .dim-badge {
  color: #212529;
  background-color: #dae0e5;
}
.result-card:not(:hover) a {
  color: var(--dark);
}
.result-card:hover .risk-of-bias {
  color: #212529;
  background-color: #dae0e5;
}

.intervention-badge {
  background-color: var(--intervention-background);
}
.outcome-badge {
  background-color: var(--outcome-background);
}
.population-badge {
  background-color: var(--population-background);
}

.article-type {
  color: #212529;
  background-color: #dae0e5;
  position: absolute;
  right: 0;
  top: 0;
  min-width: 3em;
  text-align: center;
  padding: 0 1em;
  border-bottom-left-radius: 0.25rem;
  font-size: 75%;
}

.risk-of-bias {
  position: absolute;
  bottom: 0;
  right: 0;
  padding: 0 0.5em 0.5em 0.5em;
  color: var(--gray);
  text-align: right;
  border-top-left-radius: 0.25rem;
  font-size: 75%;
}
.risk-of-bias > div {
  display: inline-block;
  margin-left: 1em;
}
.risk-of-bias > div span:first-child {
  font-weight: bolder;
}
.result-card:hover .risk-of-bias span[data-bias="l"] {
  color: var(--green);
}
.highlightText {
  background-color: yellow;
}
.row{
  align-items: stretch;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  overflow-x: auto;
  overflow-y: hidden;
}
    
.rowns{
  align-items: stretch;
  display: flex;
  flex-direction: row;
  flex-wrap: nowrap;
  overflow-x: auto;
  overflow-y: hidden;  
}
.card {
  margin-top: 2rem;
  text-align: left;
  max-width: 40%;
  max-height: 400px;
  padding: 2rem;
  margin-bottom: 2rem;
  margin-left: 1em;
  margin-right: 1em;
  // border: 1;
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  flex-wrap: nowrap;
  // overflow-x: hidden;
  overflow-y: scroll;
}


.card-text {
  font-size: 100%;
}

.card-title {
  font-size: 120%;
}
</style>
