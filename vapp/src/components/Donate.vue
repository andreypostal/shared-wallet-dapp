<template>
  <div v-if="isDrizzleInitialized">
    <b-row>
      <b-col cols="12">
        <h2>Put money into the wallet</h2>
      </b-col>
      <b-col cols="12">
        <InputAmount v-model="amount" />
      </b-col>
      <b-col cols="12">
        <b-button class="mt-2" variant="success" block @click="execute" :disabled="actionLoading">
          <b-spinner small v-if="actionLoading"></b-spinner>
          <span v-else>SEND</span>
        </b-button>
      </b-col>
      <b-col cols="12" class="mt-2">
        <b-alert v-if="showSuccessAlert" variant="success" :show="10" dismissible @dismissed="showSuccessAlert=false">
          Operation successfully received
        </b-alert>
        <b-alert  v-if="showErrorAlert" variant="warning" :show="10" dismissible @dismissed="showErrorAlert=false">
          {{ errorMessage }}
        </b-alert>
      </b-col>
    </b-row>
  </div>
</template>

<script>
import InputAmount from "./inputs/Amount.vue";
import { mapGetters } from "vuex";

export default {
  components: {
    InputAmount,
  },
  data() {
    return {
      actionLoading: false,
      amount: null,
      transactionID: null,
      transactionHash: null,
      errorMessage: "",
      showSuccessAlert: false,
      showErrorAlert: false,
    };
  },
  computed: {
    ...mapGetters("accounts", ["activeAccount"]),
    ...mapGetters("drizzle", ["drizzleInstance", "isDrizzleInitialized"]),
    utils() {
      return this.drizzleInstance.web3.utils;
    },
    transaction() {
      return this.drizzleInstance.store.getState().transactions[
        this.transactionHash
      ];
    },
    transactionSuccessfull() {
      if (!this.transactionHash) {
        return false;
      }

      return this.transaction.status == "success";
    },
  },
  methods: {
    execute() {
      try {
        if (!this.amount) {
          throw "You must enter the amount";
        }

        this.actionLoading = true;
        this.transactionHash = null;

        this.drizzleInstance.contracts.Wallet.web3.eth.sendTransaction({
          to: this.drizzleInstance.contracts.Wallet.address,
          from: this.activeAccount,
          value: this.utils.toWei(this.amount, 'Ether')
        }).then(() => {
          this.amount = null;
          this.showSuccessAlert = true;
          this.actionLoading = false;
        }).catch((e) => {
          this.actionLoading = false;
          this.showErrorAlert = true;
          this.errorMessage = e.message || e;
        });
      } catch (e) {
        this.actionLoading = false;

        this.showErrorAlert = true;
        this.errorMessage = e.message || e;
      }
    },
  },
};
</script>

<style>
</style>