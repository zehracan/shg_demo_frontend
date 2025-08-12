<template>
  <div id="app">
    <b-tabs content-class="mt-3">
      <b-tab title="Operasyon Girişi" active>
        <GocmenOperasyonFormu />
      </b-tab>
      <b-tab title="Excel Yükleme">
        <GocmenExcelYukleme />
      </b-tab>
    </b-tabs>
  </div>
</template>

<script>
import GocmenOperasyonFormu from './components/GocmenOperasyonFormu.vue';
import GocmenExcelYukleme from './components/GocmenExcelYukleme.vue';

export default {
  name: 'App',
  components: {
    GocmenOperasyonFormu,
    GocmenExcelYukleme
  }
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  padding: 20px;
}

.nav-tabs .nav-link {
  font-weight: 500;
}

.nav-tabs .nav-link.active {
  color: #007bff;
  border-color: #007bff #007bff #fff;
}
</style>