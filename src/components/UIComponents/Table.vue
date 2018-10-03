<template>
  <table class="table">
    <thead>
      <slot name="columns">
        <th v-for="column in columns">{{column}}</th>
      </slot>
    </thead>
    <tbody>
    <tr v-for="item in data" v-on:click="clickList(item)">
      <slot :row="item">
        <td v-for="column in columns" v-if="hasValue(item, column)">{{itemValue(item, column)}}</td>
      </slot>
    </tr>
    </tbody>
  </table>
</template>
<script>
  import EventBus from '../../event-bus.js'
  export default {
    name: 'l-table',
    props: {
      columns: Array,
      data: Array
    },
    methods: {
      hasValue (item, column) {
        return item[column.toLowerCase()] !== 'undefined'
      },
      itemValue (item, column) {
        return item[column.toLowerCase()]
      }, 
      clickList(item) {
        console.log(item.id)
        EventBus.$emit('selected-unit-id', item.id)
      }
    }
  }
</script>
<style>
</style>
