<template>
    <div>
        <b-form-group
            id="search_address_group"
            label="Enter Address"
            :label-for="id"
            class="required-field"
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
    </div>
</template>

<script>
/* eslint-disable */

const ADDRESS_COMPONENTS = {
  subpremise: 'short_name',
  street_number: 'short_name',
  route: 'long_name',
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
      types: 'address'
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
        this.$emit('placechanged', this.formatResult(place), place, this.id)
        this.autocompleteText = document.getElementById(this.id).value
      }
    },
    updateGeolocation (callback = null) {
      if (navigator.geolocation) {
        let options = {}

        navigator.geolocation.getCurrentPosition(position => {
          let geolocation = {
            lat: position.coords.latitude,
            lng: position.coords.longitude
          }

          this.geolocation.loc = geolocation
          this.geolocation.position = position

          if (callback) callback(geolocation, position)
        }, err => {
          this.$emit('error', 'Cannot get Coordinates from navigator', err)
        }, options)
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
    filterGeocodeResultTypes (results) {
      if (!results || !this.types) return results

      let output = []
      let types = [this.types]

      if (types.includes('(cities)')) types = types.concat(CITIES_TYPE)
      if (types.includes('(regions)')) types = types.concat(REGIONS_TYPE)

      for (let r of results) {
        for (let t of r.types) {
          if (types.includes(t)) {
            output.push(r)
            break
          }
        }
      }

      return output
    }
  }
}
</script>
