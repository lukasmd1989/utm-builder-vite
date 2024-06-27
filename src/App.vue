<template>
<router-view />
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
name: 'App',
});
</script>

<style>
/* Global styles */
body {
font-family: Arial, sans-serif;
background-color: #f5f5f5;
margin: 0;
padding: 0;
}
</style>