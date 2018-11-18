<template>
  <header>
    <h1>Profile Viewer</h1>
    <div v-if="session">
      <p id="webId">{{session.webId}}</p>
      <button @click="$emit('logout')">Logout</button>
    </div>
    <button v-else @click="$emit('login')">Login</button>
  </header>
</template>

<script>
export default {
  name: 'app-header',
  props: {
    session: Object
  }
}
</script>

<style scoped>
  header {
    display: flex;
    flex-flow: row wrap;
    justify-content: space-between;
    align-items: center;
  }

  header > * {
    margin: 0 16px;
  }

  #webId {
    display: inline-block;
    margin: 0 16px;
  }
</style>
