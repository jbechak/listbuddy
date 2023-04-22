<template>
  <div class="home">
    <buddy-home />
  </div>
</template>

<script>
// @ is an alias to /src
import BuddyHome from '@/components/BuddyHome.vue'

export default {
  name: 'HomeView',
  components: {
    BuddyHome
  }
}
</script>
