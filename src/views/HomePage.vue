<template>
  <div>
    <b-container>
      <b-row>
        <b-col>
          <b-form-input v-model="keyword" placeholder="Masukkan nama kecamatan"></b-form-input>
        </b-col>
      </b-row>
      <b-row>
        <b-col>
          <b-table
            :items="locations"
            empty-text="Masukkan minimal 3 karakter untuk menampilkan hasil"
            show-empty
            striped
            hover
          ></b-table>
          <b-pagination
            v-if="locations.length > 0"
            v-model="page"
            :total-rows="rows"
            :per-page="pageSize"
          ></b-pagination>
        </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import axios from 'axios'
import {debounce} from '@/helpers/debounce.js'

export default {
  name: 'HomePage',
  data() {
    return {
      keyword: '',
      page: 1,
      pageSize: 10,
      rows: 0,
      locations: []
    }
  },
  methods: {
    async fetchLocation (isInit = false) {
      try {
        if (isInit) this.page = 1
        const query = {
          page: this.page,
          page_size: this.pageSize,
          q: this.keyword
        }
        let queries = []
        for (const key in query) {
          queries.push(`${key}=${query[key]}`)
        }
        const { data } = await axios.get(`https://api.dev.scalev.id/v1/location?${queries.join('&')}`)
        if (data.code === 200) {
          this.locations = data.data.results
          this.rows = data.data.count
        } else {
          throw data
        }
      } catch (error) {
        console.error(error)
      }
    }
  },
  watch: {
    page () {
      this.fetchLocation()
    },
    keyword: debounce(function(value) {
      if (value.length > 2) {
        this.fetchLocation(true)
      } else this.locations = []
    }, 500)
  }
}
</script>
