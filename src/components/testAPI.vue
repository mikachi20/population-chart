<template>
  <h1 id="title">都道府県別人口チャート</h1>
  <div>{{ isFirst }}</div>
  <h3>一つ以上選択してください</h3>
  <div id="checkList">
    <div
      id="choice-content"
      v-for="(prefecture, index) in prefectures"
      :key="index"
    >
      <input type="checkbox" v-bind:value="index" v-model="checkedValues" />{{
        prefecture.prefName
      }}
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      isFirst: "",
      prefectures: [],
      population: [],
      checkedValues: [],
      viewPopulation: [],
    };
  },
  // 都道府県の取得, 既に実施済みなら実行しない
  created: async function () {
    if (localStorage.prefectures !== undefined) {
      let pre = JSON.parse(localStorage.prefectures);
      this.prefectures = pre;
    } else {
      console.log("prefecturesData取得");
      let res = await fetch(
        "https://opendata.resas-portal.go.jp/api/v1/prefectures",
        {
          method: "GET",
          mode: "cors",
          headers: {
            "X-API-KEY": "8wC5jXh3N5ImNHBjezuCaxCVxt07SdRO13K5sUes",
          },
        }
      );
      res = await res.json();
      this.prefectures = res.result;
      localStorage.prefectures = JSON.stringify(this.prefectures);
    }
    if (localStorage.populationData === undefined) {
      this.isFirst = "初期データ作成中。。。(少々お待ちください)";
      console.log("poplulationDataの作成");
      localStorage.populationData = JSON.stringify();
      for (let i = 0; i < 48; i++) {
        let res = await fetch(
          "https://opendata.resas-portal.go.jp/api/v1/population/composition/perYear?cityCode=-&prefCode=" +
            i,
          {
            method: "GET",
            mode: "cors",
            headers: {
              "X-API-KEY": "8wC5jXh3N5ImNHBjezuCaxCVxt07SdRO13K5sUes",
            },
          }
        );
        res = await res.json();
        res = {
          data: res.result.data[0].data,
        };
        this.population.push(res);
        res = null;
      }
      console.log("Data" + this.population);
      localStorage.populationData = JSON.stringify(this.population);
      this.isFirst = "作成完了";
    } else {
      this.population = JSON.parse(localStorage.populationData);
      console.log("通信済み");
      console.log(this.population);
    }
  },
};
</script>

<style>
#checkList {
  display: flex;
  flex-wrap: wrap;
}

#checkList input[type="checkbox"] {
  flex-basis: 25%;
  margin-right: 5px;
  margin: 20px;
}
</style>
