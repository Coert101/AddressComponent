<template>
    <div>

        <b-container fluid>
            <b-row>
                <b-col>
                    <b-form-group
                        id="search_address_group"
                        label="Enter Address"
                        :label-for="id"
                    >
                        <b-form-input
                            size="lg"
                            ref="autocomplete"
                            type="text"
                            :id="id"
                            v-model="autocompleteText"
                        >
                        </b-form-input>
                    </b-form-group>
                </b-col>
            </b-row>
            <b-row>
                <b-col cols="3">
                    <b-form-group
                        id="address_number_group"
                        label="Street Number"
                        label-for="address_number"
                    >
                        <b-form-input
                            size="lg"
                            type="text"
                            id="address_number"
                            v-model="addressData.streetNumber"
                        >
                        </b-form-input>
                    </b-form-group>
                </b-col>
                <b-col cols="9">
                    <b-form-group
                        id="address_street_group"
                        label="Street"
                        label-for="address_street"
                    >
                        <b-form-input
                            size="lg"
                            type="text"
                            id="address_street"
                            v-model="addressData.street"
                        >
                        </b-form-input>
                    </b-form-group>
                </b-col>
            </b-row>
             <b-row>
                <b-col>
                    <b-form-group
                        id="address_suburb_group"
                        label="Suburb"
                        label-for="address_suburb"
                    >
                        <b-form-input
                            size="lg"
                            type="text"
                            id="address_suburb"
                            v-model="addressData.suburb"
                        >
                        </b-form-input>
                    </b-form-group>
                </b-col>
            </b-row>
            <b-row>
                <b-col>
                    <b-form-group
                        id="address_city_group"
                        label="City"
                        label-for="address_city"
                    >
                        <b-form-input
                            size="lg"
                            type="text"
                            id="address_city"
                            v-model="addressData.city"
                        >
                        </b-form-input>
                    </b-form-group>
                </b-col>
            </b-row>
            <b-row>
                <b-col>
                    <b-form-group
                        id="address_province_group"
                        label="Province"
                        label-for="address_province"
                    >
                        <b-form-input
                            size="lg"
                            type="text"
                            id="address_province"
                            v-model="addressData.province"
                        >
                        </b-form-input>
                    </b-form-group>
                </b-col>
            </b-row>
            <b-row>
                <b-col>
                    <b-form-group
                        id="address_country_group"
                        label="Country"
                        label-for="address_country"
                    >
                        <b-form-input
                            size="lg"
                            type="text"
                            id="address_country"
                            v-model="addressData.country"
                        >
                        </b-form-input>
                    </b-form-group>
                </b-col>
            </b-row>
            <b-row>
                <b-col>
                    <b-form-group
                        id="postal_code_group"
                        label="Postal Code"
                        label-for="postal_code"
                    >
                        <b-form-input
                            size="lg"
                            type="text"
                            id="postal_code"
                            v-model="addressData.postalCode"
                        >
                        </b-form-input>
                    </b-form-group>
                </b-col>
            </b-row>
        </b-container>
    </div>
</template>

<script>
/* eslint-disable */

const ADDRESS_COMPONENTS = {
  subpremise: 'short_name',
  street_number: 'short_name',
  route: 'long_name',
  sublocality_level_1: 'long_name',
  locality: 'long_name',
  administrative_area_level_1: 'short_name',
  administrative_area_level_2: 'long_name',
  country: 'long_name',
  postal_code: 'short_name'
}

const CITIES_TYPE = ['locality', 'administrative_area_level_3']
const REGIONS_TYPE = ['locality', 'sublocality', 'postal_code', 'country', 'administrative_area_level_1', 'administrative_area_level_2']

export default {
  name: 'AddressAutoFill',
  props: {
    // id: {
    //   type: String,
    //   // required: true
    //   default: 'this-id'
    // },
    // types: {
    //   type: String,
    //   default: 'address'
    // },
    // country: {
    //   type: [String, Array],
    //   default: 'za'
    // }
  },
  data () {
    return {
      autocomplete: null,
      autocompleteText: '',
      geolocation: {
        geocoder: null,
        loc: null,
        position: null
      },
      country: 'za',
      id: 'this-component',
      types: 'address',
      addressData: {
          streetNumber: 0,
          street: '',
          suburb: '',
          city: '',
          province: '',
          country: '',
          postalCode: ''
      }
    }
  },
  watch: {
    autocompleteText: function (newVal, oldVal) {
      this.$emit('inputChange', { newVal, oldVal }, this.id)
    },
    country: function (newVal, oldVal) {
      this.autocomplete.setComponentRestrictions({
        country: this.country === null ? [] : this.country
      })
    }
  },
  mounted: function () {
    const options = {}

    if (this.types) {
      options.types = [this.types]
    }

    if (this.country) {
      options.componentRestrictions = {
        country: this.country
      }
    }

    this.autocomplete = new google.maps.places.Autocomplete(
      document.getElementById(this.id),
      options
    )

    this.autocomplete.addListener('place_changed', this.onPlaceChanged)
  },
  methods: {
    onPlaceChanged () {
      let place = this.autocomplete.getPlace()

      if (!place.geometry) {
        this.$emit('no-results-found', place, this.id)
        return
      }
      if (place.address_components !== undefined) {
        this.processFormattedResults(this.formatResult(place))
      }
    },
    formatResult (place) {
      let returnData = {}

      for (let i = 0; i < place.address_components.length; i++) {
        let addressType = place.address_components[i].types[0]

        if (ADDRESS_COMPONENTS[addressType]) {
          let val = place.address_components[i][ADDRESS_COMPONENTS[addressType]]
          returnData[addressType] = val
        }
      }

      returnData['latitude'] = place.geometry.location.lat()
      returnData['longitude'] = place.geometry.location.lng()

      return returnData
    },
    processFormattedResults (formattedRes) {
      this.addressData.streetNumber = formattedRes.street_number
      this.addressData.street = formattedRes.route
      this.addressData.suburb = formattedRes.sublocality_level_1
      this.addressData.city = formattedRes.locality
      this.addressData.province = formattedRes.administrative_area_level_1
      this.addressData.country = formattedRes.country
      this.addressData.postalCode = formattedRes.postal_code
    }
  }
}
</script>
