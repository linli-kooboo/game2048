<template>
  <Page>
    <StackLayout backgroundColor="#3c495e">
      <FlexboxLayout class="toolbar" alignItems="center" justifyContent="space-between">
        <Label :text="'Score: '+score"/>
        <Label text="pause" @tap="$navigateTo(AppPage, { props:{ msg: 'Continue', gridText: gridText, score: score } })"/>
      </FlexboxLayout>
      <StackLayout>
        <GridLayout
          class="game-box"
          columns="*,*,*,*"
          rows="100,100,100,100"
          @swipe="handleSwipe"
        >
          <Label
            v-for="(val,index) in grid"
            :key="index"
            :text="gridText[index]"
            :row="val[0]"
            :col="val[1]"
            class="grid"
            :class="{active: gridText[index]}"
          />
        </GridLayout>
      </StackLayout>
    </StackLayout>
  </Page>
</template>

<script >
import App from './App';

const dialogs = require('tns-core-modules/ui/dialogs');
const fileSystemModule = require("file-system");

export default {
  data() {
    return {
      score: 0,
      AppPage: App,
      grid: Array(0,1,2,3).reduce((res,v) => ( res.push(...Array.from(Array(4),(m,n)=>[v,n])), res ),[]),
      gridText: Array(16).fill('')
    }
  },
  components: {
    App
  },
  props: {
    name: { require: true },
    HistoryGridText: null,
    HistoryScore: 0
  },
  methods: {
    creat(num) {
      let vaildPosition = this.gridText.reduce((res,v,i)=>(!v&&res.push(i),res),[]);
      let newPosition = vaildPosition[Math.round(Math.random()*vaildPosition.length)];
      this.$set(this.gridText,newPosition,num || (Math.random() > 0.3 ? 2 : 4));
    },
    group({ arr, reverse=false, type='row', num=4 }) {
      let result = [];
      let rowResult = [];
      while(arr.length) rowResult.push(arr.splice(0,num));
      if(type === 'row') result = rowResult;
      if(type === 'col') while(rowResult[0].length) result.push(rowResult.map(v => v.shift()));
      if(result.length) return reverse ? result.map(v => v.reverse()) : result;
      console.error('Incoming grouping types are incorrect!');
    },
    combination({ arr, reverse=false, type='row', num=4 }) {
      let colResult = [];
      if(type === 'row') return arr.reduce((res,v) => ( res = res.concat(reverse ? v.reverse() : v), res ),[]);
      if(type === 'col') {
        while(arr[0].length) colResult.push(arr.map(v => v.shift()));
        return (reverse ? colResult.reverse() : colResult).reduce((res,v) => ( res = res.concat(v), res ),[]);
      }
      console.error('Incoming grouping types are incorrect!');      
    },
    confirmation(res) {
      confirm({
        title: res ? "Congratulation!" : "Game over",
        message: `Your score is ${this.score}`,
        okButtonText: "OK"
      }).then(result => {
        this.$navigateTo(App);
      });
      this.AddRanking({ name: this.name, score: this.score});
    },
    AddRanking(obj) {
      const fileName = "Ranking.json";
      const file = fileSystemModule.knownFolders.documents().getFile(fileName);
      let dataRead = JSON.parse(file.readTextSync() || JSON.stringify([]));
      let currentIndex = dataRead.findIndex(v => v.name === obj.name);
      if(currentIndex === -1) {
        dataRead.push(obj)  
      } else {
        dataRead[currentIndex].score = dataRead[currentIndex].score > obj.score ? dataRead[currentIndex].score : obj.score; 
      }
      file.writeText(JSON.stringify(dataRead.sort((a,b) => (b.score - a.score))));
    },
    move(arr) {
      let step = 0, sign = 0;
      arr.map((v, i) => {
        v ? sign++ : step++;
        if(v && step) {
          arr[i-step] = v;
          arr[i] = '';
        }
        if(sign === 2){
          sign = arr[i-step-1] === arr[i-step] ? 0 : 1;
          if(arr[i-step-1] === arr[i-step]) {
            arr[i-step-1] *= 2;
            arr[i-step] = '';
            this.score += arr[i-step-1]; 
            if( arr[i-step-1] === 2048 ) this.confirmation(true);
            step++;
          }
        }
      })
      return arr;
    },
    handleSwipe(args) {
      if(args.direction === 2) {
        this.gridText = this.combination({
          arr: this.group({ arr: this.gridText }).map(row => this.move(row))
        });
      }
      if(args.direction === 1) {
        this.gridText = this.combination({
          arr: this.group({ arr: this.gridText, reverse: true }).map(row => this.move(row)),
          reverse: true
        });
      }
      if(args.direction === 4) {
        this.gridText = this.combination({
          arr: this.group({ arr: this.gridText, type: 'col' }).map(row => this.move(row)),
          type: 'col'
        });
      }
      if(args.direction === 8) {
        this.gridText = this.combination({
          arr: this.group({ arr: this.gridText, reverse: true, type: 'col' }).map(row => this.move(row)), 
          reverse: true,
          type: 'col'
        });
      }
      if( this.gridText.findIndex(x => !x) === -1 ) this.confirmation(false);              
      this.creat();
    }
  },
  mounted(){
    if(this.HistoryGridText) {
      this.gridText = this.HistoryGridText;
      this.score = this.HistoryScore;
    } else {
      this.creat(2);
      this.creat(2);
    }
  }  
}
</script>

<style scoped>
  .toolbar {
    height: 200px;
    padding: 20px 50px;
    font-size: 20px;
    font-weight: 700;
    background-color: #53ba82;
    color: #ffffff;    
  }
  .game-box {
    padding: 15px;
    background-color: #289062;
  }
  .game-box .grid {
    font-size: 30px;
    font-weight: 700;
    padding: 80px 0;
    text-align: center;
    color: #ffffff;
    margin: 10px;
    background-color: #43b883;
    border-radius: 20px;
  }
  .game-box .grid.active {
    background-color: #7cdc94;
  }
</style>
