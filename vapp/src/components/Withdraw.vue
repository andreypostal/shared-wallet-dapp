<template>
  <div v-if="isDrizzleInitialized">
    <b-row>
      <b-col cols="12">
        <h2>Withdraw money</h2>
      </b-col>
      <b-col cols="12" md="6">
        <InputAddress v-model="address" />
      </b-col>
      <b-col cols="12" md="6">
        <InputAmount v-model="amount" />
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
import InputAmount from "./inputs/Amount.vue";
import { mapGetters } from "vuex";
import InputAddress from "./inputs/Address.vue";

export default {
  components: {
    InputAmount,
    InputAddress
},
  data() {
    return {
      actionLoading: false,
      amount: null,
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
        if (!this.address || !this.amount) {
          throw "You must enter the amount and the address";
        }

        this.actionLoading = true;

        this.transactionID = this.drizzleInstance.contracts.Wallet.methods.withdraw.cacheSend(this.address, this.utils.toWei(this.amount, "Ether"));

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
              this.amount = null;

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