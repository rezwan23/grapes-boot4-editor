<script>
import Vue from 'vue';
import Ggs from '@/ggs.vue';

export default Vue.extend({
  name: 'ServeDev',
  components: {
    Ggs
  }
});
</script>

<template>
  <div id="app">
    <ggs apiEndPoint="//testapp.test/api"/>
  </div>
</template>
