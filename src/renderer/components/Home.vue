<template>
  <div class="home">
    <div class="draggable-header">

    </div>
    <div class="options-holder">
      <span @click="clearList">Clear list</span>
    </div>

    <div class="list" id="list" :class="{ 'cleared':listClearFlag }">
      <transition name="slideDown" mode="out-in" appear>
          <transition-group name="slideIn" appear mode="out-in" once>
            <div class="element" v-for="el in list" :key="el.value">
              <span class="timeTag">{{el.date}}</span>
              <span class="value" @click="useElement(el.value)">{{el.value}}</span>
            </div>
          </transition-group>
        </transition>
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
      currenciesRates: [],
      currenciesNames: [],
      currencyBase: 'EUR',
      evalText: '',
      listClearFlag: false,
    }
  },
  watch: {
    evalText: function(val){
      this.fastCalc = eval(val);
    },
    currentInput: function(val) {
      this.calculateInput();
    }
  },
  methods: {
    clearList: function(){
      this.list = [];
      this.listClearFlag = true;
      setTimeout(function() {
        this.listClearFlag = false;
      }.bind(this), 200);

    },
    addCalculation: function() {
      const dateC = new Date();

      if(this.currentInput === '') return;
      this.list.push({
        value: this.fastCalc,
        date: dateC.getHours()+":"+dateC.getMinutes()
      });

      this.currentInput = '';
      setTimeout(function(){
        let list = document.getElementById('list');
        list.scrollTop = list.scrollHeight;
      },1);
    },
    useElement: function(value) {
      this.currentInput += value;
      document.getElementById('mainInput').focus();
    },
    updateCurrencies: function(base=null) {
      axios.get(`https://api.exchangeratesapi.io/latest?base=EUR`)
        .then((res)=>{
          if(res.status===200){
            this.currenciesRates[res.data.base] = res.data.rates;
            for(let currency of this.parseCurrenciesNames(res.data.base)) {
              axios.get(`https://api.exchangeratesapi.io/latest?base=${currency}`)
                .then((res)=>{
                  if(res.status===200){
                    this.currenciesRates[res.data.base] = res.data.rates;
                  }
                })
            }
          }
        })
    },
    calculateInput: function() {

      let tempText = this.currentInput;

      tempText = tempText.toLowerCase();

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

      tempText = tempText.replace(/\((.*?)[\\\/](.*?)\)/, function(a,b,c){
        c = c.toUpperCase();
        b = b.toUpperCase();
        if(this.currenciesRates.hasOwnProperty(b) && this.currenciesRates.hasOwnProperty(c)){
          return `${this.currenciesRates[b][c]}`;
        }

      }.bind(this));

      this.evalText = tempText;
    },
    parseCurrenciesNames: function(base) {
      for(let currency in this.currenciesRates[base]){
        this.currenciesNames.push(currency);
      }
      return this.currenciesNames;
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
.slideIn-enter-active {
  transition: all .3s ease;
}
.slideIn-leave-active {
  transition: all .3s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slideIn-enter, .slideIn-fade-leave-to
{
  transform: translateX(10px);
  opacity: 0;
  span {
    font-size: 0px;
  }
}

.slideDown-enter-active {
  transition: all .3s ease;
}
.slideDown-leave-active {
  transition: all .3s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}
.slideDown-enter, .slideDown-fade-leave-to
{
    transform: translateY(100px);
    opacity: 1;
}

.home {
  width: 100vw;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  flex-grow: 1;
  height: 100vh;

  .options-holder {
    display: flex;
    position: fixed;
    top: 0;
    right: 0;
    transform: translate(-10px, 30px);

    span {
      color: #AAA;
      padding: 5px 8px 3px 8px;
      font-size: 14px;
      transition: 0.2s ease;
      border-radius: 30px;
      line-height: 15px;
      cursor: pointer;

      &:hover {
        color: #FFF;
        background: #ff6b81;
      }
    }
  }

  .list {
    flex-grow: 1;
    width: 100%;
    height: 100%;
    display: flex;
    background: #27272A;
    overflow: scroll;
    padding-top: 20px;
    flex-direction: column;
    // justify-content: flex-end;
    transition: all 0.3s ease;
    scroll-behavior: smooth;

    &.cleared {
      .element {
        opacity: 0;
        filter: blur(10px);
      }
    }

    .element {
      padding: 8px 15px;
      opacity: 1;
      filter: none;
      transition: all 0.2s ease;
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
