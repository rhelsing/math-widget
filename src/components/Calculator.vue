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

function rawEval(lines){
  let result = []
  let scope = {}
  lines.forEach(function(e){
    result.push( math.eval(e, scope) )
  });
  return [result, scope]
}


export default {
  name: 'Calculator',
  props: {
    msg: String
  },
  mounted: function(){
    this.evaluate()
    var vm = this;
    this.intervalid1 = setInterval(function(){
        vm.evaluate()
    }, 500);
  },
  methods: {
    evaluate: function(){
      const raw = document.getElementById('calc').innerText
      let lines = raw.split('\n').map(x => x.trim() ).filter(x => x.length > 0 )
      //build up live stock quotes: https://api.iextrading.com/1.0/stock/aapl/price
      let processedQuotes = this.processQuotes(lines).then(function(data){
        Promise.all(data).then(function(results) {
          let result = rawEval(results)
          document.getElementById('results').innerText = result[0].map(x => x.toFixed(2)).join('\n')
        })
      })
    },
    processQuotes: async function(lines){
      //replace each line w/ @quote w/ price
      return await lines.map(async function(x){
        if(x.includes('@')){
          let matches = x.match(/[@][a-z]{1,4}/g)
          let mapped = await matches.map(async function(ticker){
            let priceObj = await fetch('https://api.iextrading.com/1.0/stock/'+ticker.replace('@', '')+'/price')
            let data = await priceObj.json()
            return [ticker, data]
          });
          return Promise.all(mapped).then(function(results) {
            let tString = x
            results.forEach(function(elm){
              tString = tString.replace(elm[0], elm[1])
            })
            return tString;
          });
        }else{
          return x
        }
      });

    },
    rawEval: function(lines){
      let result = []
      let scope = {}
      lines.forEach(function(e){
        result.push( math.eval(e, scope) )
      });
      return [result, scope]
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
