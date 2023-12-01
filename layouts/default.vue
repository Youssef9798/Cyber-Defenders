<template>
  <div class="flex flex-col w-full min-h-screen">
    <GlobalHeader />
    <div class="flex w-full h-full">
      <nuxt-child />
    </div>
  </div>
</template>

<script>
import GlobalHeader from '~/components/global/Header.vue'
export default {
  components: {
    GlobalHeader,
  },
}
</script>
