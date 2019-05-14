<template>
  <Page>
    <ActionBar title="Ranking List of 2048"/>
    <StackLayout backgroundColor="#3c495e">
      <Label :text="Ranking.info" color="#ffffff" />      
      <GridLayout
        class="row"
        v-for="(item,index) in Ranking"
        :key="index"
        columns="*,*"
        rows="auto"
        :class="{ active: index < 3 }"
      >
        <Label :text="item.name" row="0" col="0" />
        <Label :text="item.score" row="0" col="1" />
      </GridLayout>
    </StackLayout>
  </Page>
</template>

<script >
const fileSystemModule = require("file-system");

export default {
  computed: {
    Ranking() {
      const fileName = "Ranking.json";
      const file = fileSystemModule.knownFolders.documents().getFile(fileName);
      const jsonDataRead = file.readTextSync() || JSON.stringify({info: 'No ranking information yet.'});
      return JSON.parse(jsonDataRead);
    }    
  }
}
</script>

<style scoped>
  .row {
    text-align: center;
    color: #ffffff;
    margin: 10px;
    font-size: 24px;
    font-weight: 700;
  }
  .row.active {
    color: #43b883;
  }
</style>
