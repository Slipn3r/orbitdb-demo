<template>
  <div class="dropdown">
    <span>Хабр: </span>
    <dropdown :options="options" :selected="active" v-on:updateOption="onSelect"></dropdown>
    <p>Последнее обновление: {{lastUpdated}}</p>
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
      const ipfsOptions = {
        repo: './ipfs',
        EXPERIMENTAL: {
          pubsub: true,
        },
        config: {
          Addresses: {
            Swarm: [
              '/dns4/wrtc-star1.par.dwebops.pub/tcp/443/wss/p2p-webrtc-star/',
              '/dns4/wrtc-star2.sjc.dwebops.pub/tcp/443/wss/p2p-webrtc-star/',
              '/dns4/webrtc-star.discovery.libp2p.io/tcp/443/wss/p2p-webrtc-star/'
            ]
          },
        }
      }
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
        const options = {
          accessController: {
            write: ['*']
          }
        }

        this.db = await orbitdb.keyvalue('first-database', options)
        this.address = this.db.address.toString()
        console.log('created database at ' + this.address)
        await this.db.put('value', 'торт', { pin: true })
        await this.db.put('lastUpdated', new Date().toLocaleString(), { pin: true })
        this.getData()
      } catch (e) {
        console.error(e)
      }
    },
    openDatabase: async function (orbitdb) {
      try {
        this.db = await orbitdb.keyvalue(this.address)
        console.log('opened database at ' + this.address)
        console.log(this.db)
        this.getData()
      } catch (e) {
        console.error(e)
      }
    },
    getData() {
      setInterval(() => {
        try {
          const value = this.db.get('value')
          this.active.name = value
          const lastUpdated = this.db.get('lastUpdated')
          this.lastUpdated = lastUpdated
        } catch (e) {
          console.error(e)
        }
      }, 3000);
    },
    updateDatabase: async function () {
      try {
        await this.db.put('value', this.active.name, { pin: true })
        await this.db.put('lastUpdated', this.lastUpdated, { pin: true })
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
