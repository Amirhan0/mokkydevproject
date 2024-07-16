<script setup>
import { inject } from 'vue'

const toggleFavorites = inject('toggleFavorites')
const props = defineProps({
  id: Number,
  image: String,
  name: String,
  price: Number,
  isAdded: Boolean,
  isFavorite: Boolean,
  onClickAdd: Function,
  onClickFavorite: Function
})

const onClickFavorites = () => {
  toggleFavorites(props.id)
}
</script>

<template>
  <div class="border p-10 rounded-xl relative transition hover:-translate-y-2 hover:shadow-xl">
    <img
      @click="onClickFavorites"
      :src="isFavorite ? '/like-2.svg' : '/like-1.svg'"
      class="absolute top-5 left-5 cursor-pointer"
      alt=""
    />
    <img :src="image" alt="" />
    <p class="mt-5 mb-3">{{ name }}</p>
    <div class="flex justify-between">
      <div class="flex flex-col">
        <span class="uppercase text-gray-400">цена:</span>
        <span class="font-bold">{{ price }} тенге</span>
      </div>
      <img @click="onClickAdd" :src="isAdded ? 'checked.svg' : 'plus.svg'" alt="" />
    </div>
  </div>
</template>
