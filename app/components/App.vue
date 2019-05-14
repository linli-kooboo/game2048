<template>
  <Page>
    <ActionBar title="Welcome to 2048!"/>
    <DockLayout stretchLastChild="false" backgroundColor="#3c495e">
      <StackLayout dock="bottom">
        <Label :text="startMsg" @tap="handleStart" class="btn" />
        <Label text="Restart" @tap="handleRestart" class="btn" />
        <Label text="Ranking List" @tap="handleRank" class="btn" />
      </StackLayout>
    </DockLayout>
  </Page>
</template>

<script >
import Game from './Game';
import RankingList from './RankingList';

const dialogs = require('tns-core-modules/ui/dialogs')

export default {
  data() {
    return {
      startMsg: this.msg || 'Start'
    }
  },
  props: {
    msg: '',
    gridText: null,
    score: 0
  },
  components: {
    Game,
    RankingList
  },
  methods: {
    handleStart(name) {
      if(this.startMsg === 'Continue') {
        this.$navigateTo(Game, { props: {
          name: name,
          HistoryGridText: this.gridText,
          HistoryScore: this.score
        }});
      }else {
        this.handleRestart();
      }
    },
    handleRestart() {
      prompt({
        message: "Please enter your name:",
        okButtonText: "OK",
        cancelButtonText: "Cancel",
        inputType: dialogs.inputType.text
      }).then(result => {
        if(!result.result) return;
        if(!!result.text.trim()) {
          this.$navigateTo(Game, {
            props: {
              name: result.text
            }
          });
          this.startMsg = 'Continue';
        } else {
          this.handleStart(result.text);
        }
      });
    },
    handleRank() {
      this.$navigateTo(RankingList);
    }
  }
}
</script>

<style scoped>
  ActionBar {
    background-color: #53ba82;
    color: #ffffff;
  }

  .btn {
    height: 200px;
    line-height: 200px;
    font-weight: 700;
    color: #ffffff;
    text-align: center;
    vertical-align: middle;
    background-color: #43b883;
    border-radius: 10px;
  }
</style>
