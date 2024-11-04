<script setup lang="ts">
import { defineProps } from 'vue'

// const props = defineProps(['photos'])
// console.log(props)


type TypePhoto = {
  name: string,
  photos: string[]
}
const props = defineProps<TypePhoto>()
</script>

<template>
  <div class="collection">
    <img class="collection__big" :src="props.photos[0]" alt="Item" />
    <div class="collection__bottom">
      <img class="collection__mini" :src="props.photos[0]" alt="Item" />
      <img class="collection__mini" :src="props.photos[0]" alt="Item" />
      <img class="collection__mini" :src="props.photos[0]" alt="Item" />
    </div>
    <h4>{{ props.name }}</h4>
  </div>
</template>

