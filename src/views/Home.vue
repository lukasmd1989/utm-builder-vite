<template>
 <div class="home">
  <UTMBuilder />
 </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import UTMBuilder from '@/components/UTMBuilder.vue';

export default defineComponent({
name: 'Home',
components: {
  UTMBuilder,
},
});
</script>

<style scoped>
.home {
padding: 2rem;
display: flex;
justify-content: center;
}
</style>
