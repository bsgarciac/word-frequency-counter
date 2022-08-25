<template>
  <div class="wordFrecuencyCounter">
  <!-- Header -->
    <div class="header shadow"> 
      <img class="logo" src="../assets/lumu-color.png">
      <button class="btn lumu-button">Get FREE Account</button>
    </div>

  <!-- Input -->
    <div class="text-body">
      <h2 class="title">Word Frequency Counter</h2>
      <p>Paste a text below or upload a file and analyze how often each word appears.</p>
      <div>
        <input type="file" @change="loadTextFromFile">
        <textarea v-model="text" class="form-control text-input"  rows="5"></textarea>
        <button v-on:click="analyzeText" class="btn lumu-button" >Analyze Text</button>
      </div>

    </div>

  <!-- Bar Chart -->
    <div class="results" >
      <h2 class="title" v-if="showResults">Results</h2>
      <div id="barChart"></div>
    </div>

  </div>


</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'WordFrecuencyCounter',
  data() {
    return {
      text: "",
      showResults: false
    }
  },
  methods: {
    loadTextFromFile(ev) {
      const file = ev.target.files[0];
      const reader = new FileReader();
      reader.onload = e => {
        this.text = e.target.result
      };
      reader.readAsText(file);
    },
    analyzeText(){
      this.showResults= true;
      let ocurrences = {};
      let words = this.text.split(" ");
      for(let word of words){ // Linear complexity
        console.log("This loop counts each word")
        ocurrences[word] = ocurrences[word] ?  ocurrences[word] + 1 : 1;
      }
      let results = this.formatResults(ocurrences);
      this.showBarChart(results);
     
    },
    formatResults(ocurrences){ // This functions makes an object into a D3 array format
      let result = []
      for(let ocurrence in ocurrences){ // Linear complexity
        console.log("This loop formats the list of ocurrences into a D3 array format")

        result.push({word: ocurrence, count: ocurrences[ocurrence]})
      }
      return result 
    },
    showBarChart(results){
      let data = results.sort((a,b)  => b.count - a.count);
      // Showing svg with initial size
      d3.selectAll("#barChart  > *").remove();
      const  margin = {top: 20, right: 50, bottom: 30, left: 70}, 
        width = 800 - margin.left - margin.right, 
        height = results.length * 50
      const svg = d3.select("#barChart").append("svg")
        .attr("viewBox", `0 0 800 ${height + 50}`)
        .attr("preserveAspectRatio", "xMidYMid meet")

      // Ranges and Domain
      const  y = d3.scaleBand().rangeRound([0, height]).padding(.2), 
        x = d3.scaleLinear().rangeRound([0, width]), 
        g = svg.append("g") 
        .attr("transform", `translate(${margin.left},${margin.top})`); 
        
        g.selectAll(".bar").exit().remove()

        y.domain(data.map(d => d.word)); 
        x.domain([0, d3.max(data, d => d.count)]); 

      
        const xAxisTicks = x.ticks().filter(tick => Number.isInteger(tick)); // Remove decimal ticks


        // Showing axis x and y
        g.append("g") 
          .attr("class", "axis axis-x") 
          .attr("transform", `translate(0,${height})`) 
          .call(d3.axisBottom(x).tickValues(xAxisTicks).tickFormat(d3.format('d'))); 
        g.append("g") 
          .attr("class", "axis axis-y") 
          .call(d3.axisLeft(y)); 

        // Showing bars
        g.selectAll(".bar") 
          .data(data) 
          .enter().append("rect") 
          .attr("class", "bar") 
          .attr("x", x(0)) 
          .attr("y", d => y(d.word) ) 
          .attr("width", d => x(d.count)) 
          .attr("height", y.bandwidth()); 
      }

      

  }
}
</script>

<style scoped>
  .title{
    margin-bottom: 25px;    
  }
  .logo{
    width: 100px;
  }
  .header{
    display: flex;
    justify-content: space-between;
    background-color: white;
    padding: 30px 50px;
  }
  .lumu-button{
    background-color: #f98e00;
    color: white;
  }
  .text-body{
    margin-top:50px;
  }
  .text-input{
    max-width: 800px;
    width: 80%;
    margin: 40px auto;
  }
  .small-button-text{
    font-size: 15px;
    margin-top: 10px;
  }
  .small-button-text:hover{
    cursor: pointer;
    text-decoration: underline;
  }
  
  .results{
    margin: 50px auto 100px;
    max-width: 800px;
  }
  @media(max-width: 425px){
    .header{
      padding: 30px;
    }
  }
</style>
