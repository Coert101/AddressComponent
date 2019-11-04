<template>
  <div class="home">
    <h1 class="page-heading">Address Autocomplete</h1>
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
  @import url('https://fonts.googleapis.com/css?family=Raleway&display=swap');

  .address-container {
    padding-left: 5rem;
    padding-right: 5rem;
  }
  .page-heading {
    margin-bottom: 1rem;
    font-family: 'Raleway', sans-serif;
  }
</style>
