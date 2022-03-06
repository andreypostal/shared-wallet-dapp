<template>
  <div v-if="isDrizzleInitialized">
    <b-row>
      <b-col cols="12">
        <h2>Transfer ownership to</h2>
      </b-col>
      <b-col cols="12">
        <InputAddress v-model="address" />
      </b-col>
      <b-col cols="12">
        <b-button class="mt-2" variant="warning" block @click="execute">
          <b-spinner small v-if="actionLoading"></b-spinner>
          <span v-else>TRANSFER</span>
        </b-button>
      </b-col>
      <b-col class="mt-2" cols="12" v-if="!actionLoading && transactionHash">
        <b-alert v-if="transactionSuccessfull" variant="success" :show="10" dismissible @dismissed="transactionHash=null">
          Operation successfully received
        </b-alert>
        <b-alert v-else variant="warning" :show="10" dismissible @dismissed="transactionHash=null">
          {{ transaction.error.message || transaction.error }}
        </b-alert>
      </b-col>
      <b-col class="mt-2" cols="12" v-if="inputErrors" @dismissed="inputErrors=false">
        <b-alert :show="10" variant="warning">
          {{ inputErrorMessage }}
        </b-alert>
      </b-col>
    </b-row>
  </div>
</template>

<script>
import { mapGetters } from "vuex";
import InputAddress from "./inputs/Address.vue";

export default {
  components: {
    InputAddress
},
  data() {
    return {
      actionLoading: false,
      address: null,
      transactionID: null,
      transactionHash: null,
      inputErrors: false,
      inputErrorMessage: ""
    }
  },
  computed: {
    ...mapGetters("drizzle", ['drizzleInstance', "isDrizzleInitialized"]),
    utils() {
      return this.drizzleInstance.web3.utils;
    },
    transaction() {
      return this.drizzleInstance.store.getState().transactions[this.transactionHash]
    },
    transactionSuccessfull() {
      if(!this.transactionHash) {
        return false;
      }

      return this.transaction.status == "success";
    },
  },
  methods: {
    execute() {
      try {
        if (!this.address) {
          throw "You must enter the address";
        }

        this.actionLoading = true;

        this.transactionID = this.drizzleInstance.contracts.Wallet.methods.transferOwnership.cacheSend(this.address);

        const tmp = this.drizzleInstance.store.getState().transactionStack[this.transactionID];

        let maxTime = 120 / 0.5;
        const waiting = setInterval(() => {
          try {
            if(maxTime == 0) {
              throw "Operation expired. Something went wrong. Reload the page please.";
            }

            maxTime--;
            const hx = this.drizzleInstance.store.getState().transactionStack[this.transactionID];

            if(hx !== tmp || (this.drizzleInstance.store.getState().transactions[tmp] && this.drizzleInstance.store.getState().transactions[tmp].status == "error")) {
              this.transactionHash = hx;
              this.actionLoading = false;

              this.address = null;

              clearInterval(waiting);
            }
          } catch (e) {
            this.actionLoading = false;

            this.inputErrors = true;
            this.inputErrorMessage = e.message || e;

            clearInterval(waiting);
          }
        }, 500);

      } catch (e) {
        this.actionLoading = false;

        this.inputErrors = true;
        this.inputErrorMessage = e.message || e;
      }
    },
  },
}
</script>

<style>

</style>