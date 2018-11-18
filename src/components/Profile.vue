<template>
  <dl v-if="person">
    <template v-if="person.name">
      <dt>Name</dt>
      <dd>{{person.name}}</dd>
    </template>
  </dl>
</template>

<script>
export default {
  name: 'profile',
  props: {
    person: Object
  }
}
</script>

<style scoped>

</style>
