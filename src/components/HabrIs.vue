<template>
  <div class="dropdown">
    <span>Хабр: </span>
    <dropdown :options="options" :selected="active" v-on:updateOption="onSelect"></dropdown>
    <p>Последнее обновление: {{lastUpdated}}</p>
    <pre>{ value: {{active.name}}, lastUpdated: {{lastUpdated}} }</pre>
  </div>
</template>

<script>
import dropdown from 'vue-dropdowns'
const IPFS = require('ipfs')
const OrbitDB = require('orbit-db')
export default {
  name: 'HabrIs',
  components: {
    dropdown
  },
  data() {
    return {
      options: [{ name: 'торт' }, { name: 'не торт' }, { name: 'социальное СМИ об IT' }],
      active: {
        name: 'социальное СМИ об IT',
      },
      lastUpdated: '',
      db: false,
      //address: ''
      address: '/orbitdb/zdpuB3UcoWfRJCEeZ7rMsw3CuLjK25ahuTG1KCs9CW2b1HVsV/habr-is'
    }
  },
  methods: {
    onSelect(value) {
      this.active = value
      this.lastUpdated = new Date().toLocaleString()
      this.updateDatabase()
    },
    run: async function () {
      const ipfsOptions = { repo: './ipfs'}
      const ipfs = await IPFS.create(ipfsOptions)
      const orbitdb = await OrbitDB.createInstance(ipfs)
      if (!this.address) {
        this.createDatabase(orbitdb)
      } else {
        this.openDatabase(orbitdb)
      }
    },
    createDatabase: async function (orbitdb) {
      try {
        this.db = await orbitdb.open('habr-is', {
          create: true, 
          overwrite: true,
          localOnly: false,
          type: 'keyvalue',
          accessController: {
            write: ['*'],
          }
        })
        this.address = this.db.address.toString()
        console.log('created database at ' + this.address)
        await this.db.put('value', 'торт')
        await this.db.put('lastUpdated', new Date().toLocaleString())
        this.readDatabase()
      } catch (e) {
        console.error(e)
      }
    },
    openDatabase: async function (orbitdb) {
      try {
        this.db = await orbitdb.keyvalue(this.address, { sync: true })
        console.log('opened database at ' + this.address)
        console.log(this.db)
        this.readDatabase()
      } catch (e) {
        console.error(e)
      }
    },
    readDatabase() {
      try {
        const value = this.db.get('value')
        this.active.name = value
        const lastUpdated = this.db.get('lastUpdated')
        this.lastUpdated = lastUpdated
      } catch (e) {
        console.error(e)
      }
    },
    updateDatabase: async function () {
      try {
        await this.db.set('value', this.active.name)
        await this.db.set('lastUpdated', this.lastUpdated)
      } catch (e) {
        console.error(e)
      }
    }
  },
  mounted() {
    this.run()
  }
}
</script>
