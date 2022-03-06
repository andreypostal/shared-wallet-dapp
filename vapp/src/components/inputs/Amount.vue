<template>
  <b-input-group size="lg">
    <b-input-group-prepend is-text>
      <small><font-awesome-icon class="mr-2" icon="fa-brands fa-ethereum" /> AMOUNT</small>
    </b-input-group-prepend>
    <b-form-input type="number" v-model="inputText" />
    <b-input-group-append is-text>
      <small>ETHERS</small>
    </b-input-group-append>
  </b-input-group>
</template>

<script>
export default {
  props: {
    value: {
      default: "0",
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