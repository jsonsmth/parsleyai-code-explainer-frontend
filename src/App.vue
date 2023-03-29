<template>
  <div id="app">
    <CodeExplainer />
  </div>
</template>


<script>
import CodeExplainer from './components/CodeExplainer.vue';

export default {
  components: {
    CodeExplainer,
  },
};
</script>


<style>
body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  margin: 0;
  padding: 0;
  background-color: #f0f2f5;
}

#app {
  text-align: center;
}
</style>

