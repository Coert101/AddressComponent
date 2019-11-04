<template>
  <div class="home">
    <img alt="Vue logo" src="../assets/logo.png">
    <address-auto-fill></address-auto-fill>
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
