<template>
  <section class="container">
    <Main />
    <Footer />
  </section>
</template>

<script>
import Main from '@/components/Main.vue'
import Footer from '@/components/Footer.vue'

export default {
  name: 'App',
  components: {
    Main,
    Footer
  },
}
</script>

<style>
  :root {
    --dark: #121212;
    --light: #eee;
    --lighter-accent: #352b41;
    --accent: #d3b0fd;
    --rippleRgb: rgb(238, 238, 238);
  }

  html, body {
    background-color: var(--dark);
    height: 100%;
    margin: 0;
  }

  body {
    display: flex;
    flex-direction: column;
  }

  .container {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", "Roboto", "Helvetica Neue", 
    Arial, sans-serif, "Apple Color Emoji", "Segoe UI Emoji", "Segoe UI Symbol";
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    margin-top: 60px;
    color: var(--light);
    flex: 1 0 auto;
    display: flex;
    flex-direction: column;
  }
</style>