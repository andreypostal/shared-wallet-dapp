<template>
  <div v-if="isDrizzleInitialized" id="app">
    <Navbar />

    <b-container fluid class="mt-4">
      <b-tabs pills card vertical>
        <b-tab title="Home" active
          >
          <p class="small mb-0"><font-awesome-icon icon="fa-solid fa-id-card-clip" /> <span class="font-weight-bold ml-2">{{ activeAccount }}</span></p>
          <b-progress v-if="getBalance && getAllowance" :max="getBalance" height="2rem">
            <b-progress-bar :value="getAllowance"></b-progress-bar>
          </b-progress>
          <p class="small text-center">
            <span>My allowance: <strong>{{ getAllowance }} / {{ getBalance }} ethers</strong></span>
          </p>
          <hr />
          <Donate class="mt-5" />
          <hr />
          <Withdraw class="mt-5" />
          <br>
        </b-tab
        >
        <b-tab v-if="isOwner" title="Allowance">
          <AllowanceMethod method="setAllowance" title="Set allowance to" />
          <hr />
          <AllowanceMethod
            class="mt-4"
            method="addAllowance"
            title="Add allowance from"
          />
          <hr />
          <AllowanceMethod
            class="mt-4"
            method="subAllowance"
            title="Subtract allowance from"
          />
        </b-tab>
        <b-tab v-if="isOwner" title="Owner">
          <TransferOwner />
        </b-tab>
      </b-tabs>
    </b-container>
  </div>

  <div v-else>Loading...</div>
</template>

<script>
import Navbar from "./components/Navbar.vue";
import Donate from "./components/Donate.vue";
import Withdraw from "./components/Withdraw.vue";

import { mapGetters } from "vuex";
import AllowanceMethod from "./components/AllowanceMethod.vue";
import TransferOwner from "./components/TransferOwner.vue";

export default {
  name: "app",
  components: {
    Navbar,
    AllowanceMethod,
    Donate,
    Withdraw,
    TransferOwner
},
  computed: {
    ...mapGetters('accounts', ['activeAccount']),
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
    getAllowance() {
      this.$store.dispatch('drizzle/REGISTER_CONTRACT', {
        contractName: "Wallet",
        method: "allowance",
        methodArgs: [ this.activeAccount ],
      })

      const arg = {
        contract: "Wallet",
        method: "allowance",
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

      return this.utils.fromWei(contractData, "Ether")
    },
    getBalance() {
      const arg = {
        contract: "Wallet",
        method: "getBalance",
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

      return this.utils.fromWei(contractData, "Ether")
    },
    utils() {
      return this.drizzleInstance.web3.utils
    },
  },
  created() {
    this.$store.dispatch('drizzle/REGISTER_CONTRACT', {
      contractName: "Wallet",
      method: "isOwner",
      methodArgs: ""
    })

    this.$store.dispatch('drizzle/REGISTER_CONTRACT', {
      contractName: "Wallet",
      method: "getBalance",
      methodArgs: ""
    })
  }
};
</script>

<style>
@import url("https://fonts.googleapis.com/css2?family=Montserrat:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap");

#app {
  font-family: "Montserrat", sans-serif;
}
</style>
