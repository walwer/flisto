<template>
  <div class="home">
    <div class="draggable-header">

    </div>
    <div class="list">
      <div class="element" v-for="el in list">
        <span class="timeTag">{{el.date}}</span>
        <span class="value" @click="useElement(el.value)">{{el.value}}</span>
      </div>

    </div>
    <div class="fast-calc">
      <p class="fast-calc-text">{{ fastCalc===NaN || fastCalc==='NaN'?'':fastCalc }}</p>
    </div>
    <div class="input-holder">
        <input type="text" name="" value="" v-model="currentInput" class="input" placeholder="calc here" id="mainInput">
    </div>
</div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      list: [],
      currentInput: '',
      fastCalc: '',
      currencies: []
    }
  },
  watch: {
    currentInput: function(val) {

      let tempText = this.currentInput;
      tempText = tempText.toLowerCase();


      //square root
      tempText = tempText.replace(/sqrt\((.*?)\)/g, function(a,b){
        return `Math.sqrt(${b})`;
      });

      tempText = tempText.replace(/sin\((.*?)\)/g, function(a,b){
        return `Math.sin(${b})`;
      })

      tempText = tempText.replace(/pow\((.*?),(.*?)\)/g, function(a,b,c){
        return `Math.pow(${b},${c})`;
      });

      tempText = tempText.replace(/^([\*\/\-\+])/, function(a,b) {
        return `${this.list[this.list.length-1].value}${b}`;
      }.bind(this));

      tempText = tempText.replace(/pi/g,'Math.PI');

      this.fastCalc = eval(tempText);
    }
  },
  methods: {
    addCalculation: function(){
      const dateC = new Date();

      if(this.currentInput === '') return;
      this.list.push({
        value: this.fastCalc,
        date: dateC.getHours()+":"+dateC.getMinutes()
      });

      this.currentInput = '';
    },
    useElement: function(value) {
      this.currentInput += value;
      document.getElementById('mainInput').focus();
    },
    updateCurrencies: function(){
      axios.get('https://api.exchangeratesapi.io/latest')
        .then((res)=>{
          console.log(res);
        })
    }
  },
  created() {
    this.updateCurrencies();
  },
  mounted() {
    window.addEventListener('keyup', function(event) {
      if (event.keyCode === 13) {
        this.addCalculation();
      }
    }.bind(this));

    document.getElementById('mainInput').focus();
  }
}
</script>

<style lang="scss" scoped>
.home {
  width: 100vw;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  flex-grow: 1;
  height: 100vh;


  .list {
    flex-grow: 1;
    width: 100%;
    height: 100%;
    display: flex;
    background: #27272A;
    overflow-y: auto;
    padding-top: 20px;
    flex-direction: column;
    justify-content: flex-end;

    .element {
      padding: 8px 15px;
    }

    .timeTag {
      color: rgba(255,255,255,0.3);
      font-size: 15px;
      padding-right: 10px;
      cursor: pointer;
      user-select: none;
    }

    .value {
      color: rgba(255,255,255,0.9);
      font-size: 25px;
      cursor: pointer;
      transition: 0.2s ease;

      &:hover {
        color: #89899A;
      }
    }
  }

  .fast-calc {
    background: #232325;
    color: rgba(255,255,255,0.3);
    width: 100%;
    padding: 5px 20px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: flex-start;
    min-height: 40px;
    pointer-events: none;
    user-select: none;

    .fast-calc-text {
      font-size: 15px;
      padding-left: 20px;
    }
  }

  .input-holder {
    border-top: 1px dashed rgba(255,255,255,0.2);
    height: 80px;
    width: 100%;
    flex-grow: 0;
    flex-shrink: 0;
    background: #27272A;
    display: flex;
    flex-direction: column;
    justify-content: center;

    .input {
      font-size: 30px;
      background: none;
      border: none;
      outline: none;
      color: #999;
      padding: 0 15px;
    }
  }
}
</style>
