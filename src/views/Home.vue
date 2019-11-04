<template>
  <div class="home">
    <div class="address-container">
      <address-auto-fill></address-auto-fill>
    </div>
  </div>
</template>

<script>
import AddressAutoFill from '../components/address-component/AddressAutoFill.vue'

export default {
  name: 'home',
  components: {
    AddressAutoFill
  }
}
</script>

<style lang="scss">
  .address-container {
    padding-left: 5rem;
    padding-right: 5rem;
  }
</style>
