<template>
  <q-dialog v-model="NIP05Dialog" >
    <q-card class='flex column no-wrap' style='max-height: 90%'>
      <div class='flex row justify-end'>
        <q-btn icon="close" flat dense v-close-popup/>
      </div>
      <div class='overflow-auto'>
        <q-card-section>
          <div class="text-subtitle1 flex row overflow-auto">
            NIP05 identifier
          <a :href='NIP05Link' target='_'>{{ NIP05Link }}</a>
          </div>
          <pre v-if='NIP05Loaded'>{{ NIP05Formatted }}</pre>
          <q-inner-loading :showing="!NIP05Loaded">
            <q-spinner-orbit color="accent" size='2rem'/>
          </q-inner-loading>
        </q-card-section>
      </div>
    </q-card>
  </q-dialog>
    <q-btn
      v-if="$store.getters.isVerifiedNIP05(pubkey)"
      icon="verified"
      color="accent"
      flat
      dense
      size='xs'
      class='no-padding'
      clickable
      @click.stop="openNIP05"
    >
    <q-tooltip>
      NIP05 verified
    </q-tooltip>
  </q-btn>
</template>

<script>
import helpersMixin from '../utils/mixin'
import fetch from 'cross-fetch'
import {Notify} from 'quasar'

export default {
  mixins: [helpersMixin],
  props: {
    pubkey: {type: String, required: true},
  },

  data() {
    return {
      NIP05Dialog: false,
      NIP05Data: {},
    }
  },

  computed: {
    NIP05Link() {
      let [name, domain] = this.$store.getters
        .displayName(this.pubkey)
        .split('@')
      if (!domain) {
        domain = name
        name = '_'
      }
      return `https://${domain}/.well-known/nostr.json?name=${name}`
    },
    NIP05Formatted() {
      return this.json(this.NIP05Data)
    },
    NIP05Loaded() {
      if (Object.keys(this.NIP05Data).length) return true
      return false
    }
  },

  methods: {
    openNIP05() {
      this.loadNIP05Data()
      this.NIP05Dialog = !this.NIP05Dialog
    },

    async loadNIP05Data() {
      try {
        this.NIP05Data = await (await fetch(this.NIP05Link)).json()
      } catch (_) {
          Notify.create({
            message: 'failed to fetch NIP05 identifier',
            color: 'negative'
          })
      }
    },
  }
}
</script>

<style lang='scss' scoped>
.NIP05Frame {
  background: white;
  border: 0;
}
</style>