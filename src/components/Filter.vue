<template>
  <div class="mb-3">
    <div class="btn-group">
      <button
        v-for="(button, index) in buttons"
        v-bind:key="index"
        class="btn"
        v-bind:class="[
          filter === button ? 'btn-info' : 'btn-outline-secondary',
        ]"
        v-on:click="filterChange(button)"
      >
        {{ button }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: ["filter", "filterChange"],
  setup() {
    const buttons = ["Перемешать", "Зазеркалить"];
    return {
      buttons,
    };
  },
};
</script>
