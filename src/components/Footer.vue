<template>
  <footer>
    <span v-if="appVersion">
      v{{ appVersion }} - 
    </span>

    Made with <span class="heart">❤️</span> by 
    <a 
      href="https://github.com/paolozanchi"
      class="link"
      target="_github"
    >mmmmmeh1</a>
    |
    <a
      href="https://linkedin.com/in/paolo-zanchi"
      class="link"
      target="_linkedin"
    >
      Paolo Zanchi
    </a>
  </footer>
</template>

<script>
export default {
  name: 'Footer',
  data() {
    return {
      appVersion: process.env.VUE_APP_VERSION
    }
  }
}
</script>


<style scoped>
  footer {
    flex-shrink: 0;
    padding: .5rem;
    opacity: .8;
  }
  
  .link {
    color: var(--accent);
  }

  .heart {
    color: transparent;
    text-shadow: 0 0 0 var(--accent);
  }
</style>