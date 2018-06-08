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
    setInterval(this.evaluate(), 2000);
  },
  methods: {
    evaluate: function(){
      const raw = document.getElementById('calc').innerText
      let lines = raw.split('\n').map(x => x.trim() ).filter(x => x.length > 0 )
      //build up live stock quotes: https://api.iextrading.com/1.0/stock/aapl/price
      let processedQuotes = this.processQuotes(lines).then(function(data){
        Promise.all(data).then(function(results) {
          let result = rawEval(results)
          document.getElementById('results').innerText = result[0].join('\n')
        })
      })
    },
    processQuotes: async function(lines){
      //replace each line w/ @quote w/ price
      return await lines.map(async function(x){
        if(x.includes('@')){
          let ticker = x.split('@')[1];
          let priceObj = await fetch('https://api.iextrading.com/1.0/stock/'+ticker+'/price')
          let data = await priceObj.json()
          return x.split('=')[0].trim()+' = '+data
        }else{
          return x
        }
      });
      // var promises = lines.map(function(obj){
      //   if(obj.includes('@')){
      //     let ticker = obj.split('@')[1];
      //     return fetch('https://api.iextrading.com/1.0/stock/'+ticker+'/price').then(function(response){
      //       return response.json().then(function(data){
      //         return obj.split('=')[0].trim()+' = '+data
      //       });
      //     });
      //   }else{
      //     return obj
      //   }
      // })
      // console.log(promises)
      // Promise.all(promises).then(function(results) {
      //   return results
      // })

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
