<template>
  <b-input-group size="lg">
    <b-input-group-prepend is-text>
      <small><font-awesome-icon class="mr-2" icon="fa-solid fa-id-badge" /> ADDRESS</small>
    </b-input-group-prepend>
    <b-form-input v-model="inputText" />
  </b-input-group>
</template>

<script>
export default {
  props: {
    value: {
      type: String,
      default: "",
    },
  },
  computed: {
    inputText: {
      get() {
        return this.value;
      },
      set(value) {
        this.$emit("input", value);
      },
    },
  },

}
</script>

<style>

</style>