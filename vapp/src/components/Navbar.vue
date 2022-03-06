<template>
  <b-navbar toggleable="lg" type="dark" variant="primary">
    <b-navbar-brand href="#"><b-icon icon="wallet2"></b-icon> shared wallet</b-navbar-brand>

    <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>

    <b-collapse id="nav-collapse" is-nav>
      <b-navbar-nav class="ml-auto">
       <span class="text-secondary">
          <font-awesome-icon icon="fa-solid fa-money-bill-1" /> {{convertedBalance}} ether
        </span>
      </b-navbar-nav>
    </b-collapse>
  </b-navbar>
</template>

<script>
import { mapGetters } from 'vuex'

const capitalize = ws => ws[0].toUpperCase() + ws.slice(1).toLowerCase()

const precisionRound = (number, precision) => {
  const factor = Math.pow(10, precision)
  return Math.round(number * factor) / factor
}

export default {
  props: {
    units: {
      type: String,
      default: 'Ether'
    },
    precision: {
      type: Number,
      default: 2
    }
  },
  computed: {
    ...mapGetters('accounts', ['activeAccount', 'activeBalance']),
    ...mapGetters('drizzle', ['drizzleInstance', 'isDrizzleInitialized']),

    convertedBalance() {
      const wei = this.activeBalance
      const units = capitalize(this.units)
      return precisionRound(
        this.drizzleInstance.web3.utils.fromWei(wei, units),
        this.precision
      )
    }
  }
}
</script>

<style>

</style>