<template>
  <span 
    v-if="this.store.jobs.getActiveSlug === slug"
    class="text-inherit">
    <span class="text-inherit">detecting region</span>
  </span>
</template>

<style scoped>

</style>

<script>
import { defineComponent } from 'vue'

import { setupStore } from '@/store'

import RelayMethods from '@/shared/relays-lib.js'
import SharedComputed from '@/shared/computed.js'

import { getVisitorGeo, getClosest } from '@/utils'

export default defineComponent({
  name: 'LoadSeed',
  components: {},
  data() {
    return {
      slug: 'user/region' //REMEMBER TO CHANGE!!!
    }
  },
  setup(){
    return { 
      store : setupStore()
    }
  },
  created(){
    clearInterval(this.interval)
  },
  unmounted(){
    clearInterval(this.interval)
  },
  beforeMount(){
    this.lastUpdate = this.store.jobs.getLastUpdate(this.slug)
  },
  mounted(){
    //console.log('is processing', this.store.jobs.isJobActive(this.slug))

    if(this.store.jobs.isJobActive(this.slug))
        this.CheckRegion(true)
      else
        this.CheckRegion()

    if(!this.store.prefs.autoDetectRegion)
      this.setRefreshInterval()
  },
  updated(){},
  computed: Object.assign(SharedComputed, {}),
  methods: Object.assign({
    CheckRegion(force){
      if( !this.isExpired(this.slug, 6*60*60*1000) && !force ) 
        return

      this.queueJob(
        this.slug, 
        async () => {
          try {
            const visitorGeo = await getVisitorGeo()
            this.store.user.ip = visitorGeo.query
            this.store.prefs.region = getClosest(visitorGeo)
            this.store.jobs.completeJob(this.slug)
          }
          catch(e) { //brave 
            this.store.prefs.clientSideProcessing = true
            this.store.prefs.disableGeoDetection = true
            location.reload() 
          }
        },
        true
      )
    },

  setRefreshInterval: function(){
    clearInterval(this.interval)
    this.interval = setInterval(() => {
      if(!this.store.prefs.autoDetectRegion )
        return 

      if(!this.store.jobs.isJobActive(this.slug) && !this.isSingle)
        this.CheckRegion()
    }, 1000)
  },
}, RelayMethods),
  props: {},
})
</script>

<style scoped>
 #refresh { font-size: 12pt; color:#666; margin-bottom:15px }
 #refresh button { cursor: pointer; border-radius: 3px; border: 1px solid #a0a0a0; color:#333 }
 #refresh button:hover {color:#000;}
 #refresh button[disabled] {color:#999 !important; border-color:#e0e0e0}
</style>