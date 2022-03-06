<template>
  <div v-if="isDrizzleInitialized">
    <b-row v-if="isOwner && !isStale">
      <b-col cols="12">
        <h2>{{ title }}</h2>
      </b-col>
      <b-col cols="12" md="6">
        <InputAddress v-model="address" />
      </b-col>
      <b-col cols="12" md="6">
        <InputAmount v-model="amount" />
      </b-col>
      <b-col cols="12">
        <b-button class="mt-2" variant="info" block @click="executeAllowance" :disabled="actionLoading">
          <b-spinner small v-if="actionLoading"></b-spinner>
          <span v-else>SEND</span>
        </b-button>
      </b-col>
      <b-col class="mt-2" cols="12" v-if="!actionLoading && transactionHash">
        <b-alert v-if="transactionSuccessfull" variant="success" :show="10" dismissible @dismissed="transactionHash=null">
          Operation successfully received
        </b-alert>
        <b-alert v-else variant="warning" :show="10" dismissible @dismissed="transactionHash=null">
          {{ transaction.error.message }}
        </b-alert>
      </b-col>
      <b-col class="mt-2" cols="12" v-if="inputErrors" @dismissed="inputErrors=false">
        <b-alert :show="10" variant="warning">
          {{ inputErrorMessage }}
        </b-alert>
      </b-col>
    </b-row>
  </div>
  <div v-else>Loading...</div>
</template>

<script>
import InputAddress from "./inputs/Address.vue";
import InputAmount from "./inputs/Amount.vue";
import { mapGetters } from "vuex";

export default {
  props: {
    method: {
      type: String,
      required: true,
    },
    title: {
      type: String,
      default: ""
    }
  },
  components: {
    InputAddress,
    InputAmount,
  },
  computed: {
    ...mapGetters('contracts', ['getContractData', 'contractInstances']),
    ...mapGetters("drizzle", ['drizzleInstance', "isDrizzleInitialized"]),
    isStale() {
      return !this.contractInstances["Wallet"].synced
    },
    isOwner() {
      const arg = {
        contract: "Wallet",
        method: "isOwner",
        toUtf8: false,
        toAscii: false
      }
      
      let contractData = this.getContractData(arg)

      if (typeof contractData === 'object') {
        contractData = Object.entries(contractData)
          .filter(([key]) => /^\D/.test(key))
          .map(([key, value]) => ({ key, value }))
      }

      if(contractData == "loading") {
        return false;
      }

      return contractData
    },
    abi() {
      const di = this.drizzleInstance
      return di.contracts["Wallet"].abi.find(
        item => item.name === this.method
      )
    },
    abiInputs() {
      return this.abi.inputs
    },
    utils() {
      return this.drizzleInstance.web3.utils
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
  data() {
    return {
      address: null,
      amount: null,
      actionLoading: false,
      transactionID: null,
      transactionHash: null,
      inputErrors: false,
      inputErrorMessage: "",
    };
  },
  methods: {
    executeAllowance() {
      try {
        if(!this.amount || !this.address) {
          throw "Cannot have empty fields";
        }

        this.actionLoading = true;
        this.transactionHash = null;

        const id = this.drizzleInstance.contracts["Wallet"].methods[
          this.method
        ].cacheSend(this.address, this.utils.toWei(this.amount, "Ether"))

        this.transactionID = id;
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
    }
  },
  created() {
    this.$store.dispatch('drizzle/REGISTER_CONTRACT', {
      contractName: "Wallet",
      method: "isOwner",
      methodArgs: ""
    })
  }
};
</script>

<style>
</style>