<template>
<b-card
  v-bind:title="item.ti"
  v-bind:hover_type="hover_type"
  v-bind:searchWord="searchWord"
  class="result-card shadow-sm p-3 mb-5 bg-white rounded">

  <div
    v-if="item.article_type"
    v-bind:data-article-type="item.article_type"
    class="article-type">
    {{ item.article_type }}
  </div>

  <h6 class="card-subtitle text-muted mb-2" v-if="item.article_type == 'preprint' ||  item.article_type == 'journal article'">
    <a
      v-bind:href="`https://www.ncbi.nlm.nih.gov/pubmed/${item.pmid}`"
      target="_blank">{{ item.pmid }}</a>
    {{ item.citation }}
    <a
      v-if="item.dois && item.dois[0]"
      v-bind:href="`https://dx.doi.org/${item.dois[0]}`"
      target="_blank">{{ item.dois[0] }}</a>
  </h6>

  <h6 class="card-subtitle text-muted mb-2" v-else>
    {{item.regid}}, {{item.is_recruiting}}, {{item.is_rct}}
  </h6>


  <b-card-text>
    <div class="text-meta">
    <span v-if="item.countries && item.countries.length">
      Date Registered: {{registered_date}}
      <br />
      Study Countries: {{ item.countries.join(", ") }}
      <br />
    </span>

    <div
      v-if="item.num_randomized"
      class="num-randomized">
       <b-icon
        v-b-tooltip.hover
        title="Randomized participants; extracted using machine learning"
        icon="shuffle"
        font-scale="1" /> {{ item.num_randomized }}
    </div>

    <div
      v-if="item.target_size"
      class="num-randomized"
      font-scale="1">
       <b-icon
        v-b-tooltip.hover
        title="Target size; from the trial registry."
        icon="shuffle"
        font-scale="1" /> {{ item.target_size }}
    </div>

    
    </div>

    <b-container style="margin: 1em -15px 0 -15px;">
      <b-row v-if="item.punchline_text && item.punchline_text.length">
        <p class="mb-0"><span v-html="highlight_pop(item.punchline_text , 'ptext')">{{fixParens(p) }} {{ item.punchline_text }}</span></p>
      </b-row>
        <b-row>
          <b-col xs="1" md="4" class="pico-col" v-if="hover_type == 'pop'">
            <b-badge class="population-badge dim-badge">Population</b-badge>
            <ul>
              <li v-for="p in distinct(item.population)" :key="p">
                 <span v-html="highlight_pop(p , 'pop')">{{fixParens(p) }}</span>
              </li>
              <li v-if="!item.population.length">
                <em>None extracted</em>
              </li>
            </ul>
          </b-col>

          <b-col xs="1" md="4" class="pico-col" v-if="hover_type == 'int'">
            <b-badge class="intervention-badge dim-badge">Interventions</b-badge>
            <ul>
              <li v-for="i in distinct(item.interventions)" :key="i">
                <span v-html="highlight_pop(i , 'int')">{{ fixParens(i) }}</span>
              </li>
              <li v-if="!item.interventions.length">
                <em>None extracted</em>
              </li>
            </ul>
          </b-col>

          <b-col xs="1" md="4" class="pico-col" v-if="hover_type == 'out'">
            <b-badge class="outcome-badge dim-badge">Outcomes</b-badge>
            <ul>
              <li v-for="o in distinct(item.outcomes)" :key="o">
                <span v-html="highlight_pop(o , 'out')">{{ fixParens(o) }}</span>
              </li>
              <li v-if="!item.outcomes.length">
                <em>None extracted</em>
              </li>
            </ul>
          </b-col>
        </b-row>
        <div v-if="item.prob_low_rob"
          class="risk-of-bias"
          v-b-tooltip.hover.right
          title="As determined by RobotReviewer based on the abstract text">
            <div>
              <span>Probability of low risk of bias: </span>
              <span>{{Math.round(100*item.prob_low_rob)}}%</span>
              <!-- <span v-bind:data-bias="item.low_ac_bias ? 'l' : 'h'">{{item.low_ac_bias ? "low" : "high/unknown"}}</span> -->
            </div>
        </div>
      </b-container>
    </b-card-text>
  </b-card>
</template>


<script>
export default {
  name: "Card",
  props: ['item', 'hover_type', 'searchWord'],
  computed: {
    registered_date: function() {
      return window.moment(this.item.date_registered).format("LL")
    }
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
          return s.toLowerCase().replace(this.searchWord, '<span style="background: var(--population-background); color: white;">' + this.searchWord + '</span>')
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
  font-size: 75%;
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
</style>
