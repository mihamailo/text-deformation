<template>
  <div class="btn-group">
    <button
      v-for="(button, index) in buttons"
      v-bind:key="index"
      class="btn"
      v-bind:class="[filter === button ? 'btn-info' : 'btn-outline-secondary']"
      v-on:click="filterChange(button)"
    >
      {{ button }}
    </button>
  </div>
</template>

<script>
export default {
  props: ["filter", "filterChange", "buttons"],
};
</script>
