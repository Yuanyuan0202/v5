<template>
  <div id="app">
    <SystemTable />
  </div>
</template>

<script>
import SystemTable from './components/SystemTable.vue';

export default {
  name: 'App',
  components: {
    SystemTable
  }
};
</script>

<style>
@import './assets/styles/variables.css'; /* 導入顏色變數定義文件 */

/* 這裡可以加入全局的樣式設置 */
body, html, #app {
  margin: 0;
  padding: 0;
  font-family: Arial, sans-serif;
  background-color: var(--variable-collection-black);
  color: var(--variable-collection-white);
}
</style>

