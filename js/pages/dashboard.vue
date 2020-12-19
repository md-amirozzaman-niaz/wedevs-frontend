<template>
  <card :title="$t('dashboard')">
    <router-link :to="{ name: 'product' }">
          {{ $t('product') }}
    </router-link>
  </card>
</template>

<script>
export default {
  middleware: 'auth',

  metaInfo () {
    return { title: this.$t('home') }
  }
}
</script>
