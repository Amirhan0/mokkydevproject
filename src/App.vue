<script setup>
import Header from './components/HeaderComponent.vue'
import Cards from './components/CardsItems.vue'
import { onMounted, ref, watch, reactive, provide } from 'vue'
import axios from 'axios'
import { inject } from 'vue'
// import Drawer from './components/DrawerComponent.vue'

// Инициализация состояний
const items = ref([])
const sortBy = ref('')
const filters = reactive({
  sortBy: 'title',
  searchBy: ''
})

// Функция для загрузки элементов
const fetchItems = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }
    if (filters.searchBy) {
      params.title = filters.searchBy
    }
    const { data } = await axios.get('https://269b3b45e08bcd1a.mokky.dev/items', {
      params
    })
    items.value = data.map((obj) => ({
      ...obj,
      isAdded: false,
      isFavorite: false
    }))
  } catch (error) {
    console.log(error)
  }
}

// Функция для загрузки избранных элементов
const fetchFavorites = async () => {
  try {
    const params = {
      sortBy: filters.sortBy
    }
    if (filters.searchBy) {
      params.title = filters.searchBy
    }
    const { data: favorites } = await axios.get('https://269b3b45e08bcd1a.mokky.dev/favorites', {
      params
    })
    // Обновление состояния элементов с учетом избранных
    items.value = items.value.map((item) => {
      const favorite = favorites.find((fav) => fav.sneakerId === item.id)
      if (favorite) {
        return {
          ...item,
          isFavorite: true,
          isAdded: false
        }
      }
      return item
    })
  } catch (error) {
    console.log(error)
  }
}

// Функция для переключения избранного
const toggleFavorites = async (id) => {
  try {
    // Получение текущего состояния избранных
    const { data } = await axios.get(`https://269b3b45e08bcd1a.mokky.dev/favorites`)
    // Обновление элемента по id
    const updatedItems = items.value.map(async (item) => {
      if (item.id === id) {
        const favorite = data.find((fav) => fav.sneakerId === item.id)
        if (item.isFavorite) {
          await axios.delete(`https://269b3b45e08bcd1a.mokky.dev/favorites/${favorite.id}`)
        } else {
          const obj = { sneakerId: item.id }
          await axios.post(`https://269b3b45e08bcd1a.mokky.dev/favorites`, obj)
        }
        return {
          ...item,
          isFavorite: !item.isFavorite
        }
      }
      return item
    })
    items.value = await Promise.all(updatedItems)
  } catch (error) {
    console.log(error)
  }
}
provide('toggleFavorites', toggleFavorites)

// Функции для обработки изменений фильтров
const onChangeSelect = (event) => {
  filters.sortBy = event.target.value
}

const onChangeInput = (event) => {
  filters.searchBy = `*${event.target.value}*`
}

// Загрузка данных при монтировании компонента
onMounted(async () => {
  await fetchItems()
  await fetchFavorites()
})

// Отслеживание изменений в фильтрах и повторная загрузка данных
watch(filters, fetchItems)
</script>

<template>
  <Drawer />
  <div class="w-4/5 m-auto bg-white rounded-xl shadow-xl mt-14 pb-2">
    <Header />
    <div class="flex justify-between pr-10">
      <h2 class="font-bold text-xl p-10">Все кроссовки</h2>
      <div class="flex gap-3 items-center">
        <select
          @change="onChangeSelect"
          class="border border-gray-200 rounded-md outline-none py-2 px-2"
        >
          <option value="title">По названию</option>
          <option value="price">По возрастанию цены</option>
          <option value="-price">По убыванию цены</option>
        </select>
        <div class="relative">
          <input
            @input="onChangeInput"
            type="text"
            class="border border-gray-200 rounded-md py-2 pl-10 pr-4 focus:outline-none focus:border-gray-400"
            placeholder="Поиск..."
          />
          <div class="absolute inset-y-0 left-0 pl-3 flex items-center pointer-events-none">
            <img src="/search.svg" />
          </div>
        </div>
      </div>
    </div>

    <Cards :items="items" />
  </div>
</template>

<style scoped></style>
