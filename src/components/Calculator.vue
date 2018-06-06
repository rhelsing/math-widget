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
  mounted: function(){
    this.evaluate()
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

.container, html, body {
    height: 100%;
    margin: 0;
}
.container {
    display: flex;
}

#calc {
    flex: 100%;
    outline: none;
    padding-left: 2rem;
    padding-right: 2rem;
}
#results {
    padding-left: 2rem;
    padding-right: 2rem;
    flex: 100%;
    height: 100%;
    position: absolute;
    right: 0;
    background: #0000001a;
    color: #3ff3ba
}
</style>
