<script setup>
import { ref, computed, watch } from 'vue';

// 修改這份 YouBike 即時資訊表，並加上
// 1. 站點名稱搜尋
// 2. 目前可用車輛 / 總停車格 的排序功能
// 3. 加入分頁功能，每頁 20 筆資料

// 欄位說明:
// https://data.taipei/dataset/detail?id=c6bc8aed-557d-41d5-bfb1-8da24f78f2fb
// sno：站點代號、 sna：場站名稱(中文)、 tot：場站總停車格、
// sbi：場站目前車輛數量、 sarea：場站區域(中文)、 mday：資料更新時間、
// lat：緯度、 lng：經度、 ar：地(中文)、 sareaen：場站區域(英文)、
// snaen：場站名稱(英文)、 aren：地址(英文)、 bemp：空位數量、 act：全站禁用狀態

const uBikeStops = ref([]);
const searchInput = ref('');
const sort = ref(0); //1: 升序 2: 降序
const sortType = ref('');

// const totoalPages = ref(0);

fetch('https://tcgbusfs.blob.core.windows.net/dotapp/youbike/v2/youbike_immediate.json')
  .then(res => res.text())
  .then(data => {
    uBikeStops.value = JSON.parse(data);
    // console.log(uBikeStops.value);
  });

const timeFormat = (val) => {
  // 時間格式
  const pattern = /(\d{4})(\d{2})(\d{2})(\d{2})(\d{2})(\d{2})/;
  return val.replace(pattern, '$1/$2/$3 $4:$5:$6');
};

const filteredUBikeStops = computed(()=> {
  const filterResult = uBikeStops.value.length === 0 ? [] : uBikeStops.value.filter(item => item.sna.toLowerCase().includes(searchInput.value.toLowerCase()));
  if(sort.value){
    filterResult.sort((a, b) => {

      if(sortType.value === 'sbi'){
        // console.log(sortType.value);
        // console.log(sort.value);
        return sort.value === 1 ? a.sbi-b.sbi : b.sbi-a.sbi;

      }else if(sortType.value === 'tot'){
        // console.log(sortType.value);
        // console.log(sort.value);
        return sort.value === 1 ? a.tot-b.tot : b.tot-a.tot;
      }
    })
  }
  
  return filterResult;
});

const pagination = computed(() => {
  const pageLimit = 20;
  totoalPages.value = Math.ceil(filteredUBikeStops.value.length / pageLimit);

  console.log(totoalPages.value);
})
</script>

<template>
<div class="app">
  <p>
    站點名稱搜尋: <input type="text" v-model="searchInput">
  </p>

<nav>
  <ul class="pagination">
    <li class="page-item">
      <span class="page-link">Previous</span>
    </li>

    <li
      v-for="i in 20" :key="i"
      class="page-item">
      <span class="page-link">{{ i }}</span>
    </li>

    <li class="page-item">
      <span class="page-link" href>Next</span>
    </li>
  </ul>
</nav>


  <table class="table table-striped">
    <thead>
      <tr>
        <th>#</th>
        <th>場站名稱</th>
        <th>場站區域</th>
        <th>目前可用車輛
          <i class="fa fa-sort-asc" aria-hidden="true" @click="sort=1, sortType='sbi'"></i>
          <i class="fa fa-sort-desc" aria-hidden="true" @click="sort=2, sortType='sbi'"></i>
        </th>
        <th>總停車格
          <i class="fa fa-sort-asc" aria-hidden="true" @click="sort=1, sortType='tot'"></i>
          <i class="fa fa-sort-desc" aria-hidden="true" @click="sort=2, sortType='tot'"></i>
        </th>
        <th>資料更新時間</th>
      </tr>
    </thead>
    <tbody>
      <tr v-for="s in filteredUBikeStops" :key="s.sno">
        <td>{{ s.sno }}</td>
        <td >{{ s.sna }}</td>
        <td>{{ s.sarea }}</td>
        <td>{{ s.sbi }}</td>
        <td>{{ s.tot }}</td>
        <td>{{ timeFormat(s.mday) }}</td>
      </tr>
    </tbody>
  </table>
</div>

</template>

<style scoped>
.app {
  padding: 1rem;
}
.searchInput{
  background-color: #fa0;
}
</style>
