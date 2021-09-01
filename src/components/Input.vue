<template>
  <div class="justify-content-center mt-2 justify-content-center">
    <div class="position-relative">
      <textarea
        type="text"
        class="form-control"
        placeholder="Enter your text"
        v-model="inputValue"
        rows="8"
      />
      <button @click="setText(inputValue)" class="btn btn-primary">
        Деформировать
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: ["setText"],
  setup() {
    const inputValue = "";
  },
};
</script>

<style scoped>
button {
  position: absolute;
  left: 15px;
  bottom: 15px;
  opacity: 0.8;
}
</style>