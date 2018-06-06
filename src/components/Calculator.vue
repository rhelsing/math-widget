<template>
  <div class="container">
    <div contenteditable="true" id="calc" @keyup="evaluate">
      {{msg}}
    </div>
    <div id="results">
    </div>
  </div>
</template>

<script>
import math from 'mathjs'
export default {
  name: 'Calculator',
  props: {
    msg: String,
    varLookups: {}
  },
  methods: {
    evaluate: function(){
      const raw = document.getElementById('calc').innerText
      let lines = raw.split('\n').map(x => x.trim() ).filter(x => x.length > 0 )
      lines = this.rawEval(lines)
      document.getElementById('results').innerText = lines.join('\n')
    },
    rawEval: function(lines){
      let result = []
      let scope = {}
      lines.forEach(function(e){
        result.push( math.eval(e, scope) )
      });
      return result
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.container {
  height:100%;
    display:flex;
    display: -webkit-flex;
    flex-direction: row;
    -webkit-flex-direction: row;
    -webkit-align-content: stretch;
    align-content: stretch;
}
.container > * {
  flex: 1;
  height: 100%;
}
#calc{
  margin: 1rem;
  outline: none;
  margin: auto;
}
#results {
  background: #0000001a;
  margin: auto;
  color: #3ff3ba;
  height: 100%;
}
</style>
