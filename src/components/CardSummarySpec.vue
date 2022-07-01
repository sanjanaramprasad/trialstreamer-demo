<template>



<div class="result-card shadow-sm p-3 mb-5 rounded"  style="background-color:var(--population-background);" v-if="hover_type == 'pop' && this.searchArticlesResults && this.searchArticlesResults.length">
  
  
  <div class="row">

    <div class="card" v-for="item in this.searchArticlesResults">
      <blockquote >
        <h6 >{{ item.ti }}</h6>
      </blockquote>

      
        <h6>Population</h6>
      
  
      <ul>
        <li v-for="p in distinct(item.population)" :key="p">
                 <span v-html="highlight_pop(p , 'pop')">
                  {{fixParens(p) }}
                 </span>
              </li>
              <li v-if="!item.population.length">
                <em>None extracted</em>
              </li>
      </ul>
      
    </div>

  </div>

</div>



    
 
</template>


<script>
export default {
  name: "Card",
  props: ['searchArticlesResults', 'hover_type', 'searchWord'],
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
    highlight_pop(s, s_type) {
        if(this.searchWord === '') { return s }
        
        // when removing the .toLowerCase() your search becomes case-sensitive
        if (s_type == 'pop'){
          return s.replace(this.searchWord, '<span style="background: yellow; color: black;">' + this.searchWord + '</span>')
        }
        else if (s_type == 'int'){

          return s.toLowerCase().replace(this.searchWord, '<span style="background: var(--intervention-background); color: white;">' + this.searchWord + '</span>')
        }
        else if (s_type == 'out'){

          return s.toLowerCase().replace(this.searchWord, '<span style="background: var(--outcome-background); color: white;">' + this.searchWord + '</span>')
        }
        else if (s_type == 'ptext'){

          return s.toLowerCase().replace(this.searchWord, '<span style="background: #6f7070; color: white;">' + this.searchWord + '</span>')
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
.card {
  margin-top: 0;
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
