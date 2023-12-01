<template>
  <div class="w-full flex items-center bg-[#1e293b] min-h-[5rem]">
    <div class="container mx-auto">
      <div class="flex items-center gap-4">
        <img
          class="w-full h-full max-w-[190px] max-h-[40px]"
          src="images/cyberdefenders.svg"
        />
        <ul class="flex gap-2">
          <li v-for="link in links" :key="link.name">
            <nuxt-link class="text-white" :to="link.path">{{
              link.name
            }}</nuxt-link>
          </li>
        </ul>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  computed: {
    links() {
      return []
    },
  },
}
</script>
