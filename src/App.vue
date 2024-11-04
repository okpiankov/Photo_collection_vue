<script setup lang="ts">
import axios from 'axios'
import { computed, onMounted, ref, watch } from 'vue'
import debounce from 'lodash/debounce'
import CollectionPhoto from './components/CollectionPhoto.vue'

type TypePhoto = [
  {
    id: number
    category: number
    name: string
    photos: string[]
  },
]

const categories = [
  { name: 'Все' },
  { name: 'Море' },
  { name: 'Горы' },
  { name: 'Города' },
  { name: 'Архитектура' },
]

const collectionPhotos = ref<TypePhoto | []>([])
const searchValue = ref('')
const categoryActive = ref(0) // 0 будут 'Все' категории по умолчанию
const isLoading = ref(false)
const page = ref(1) // 1 т.к. первая страница и необходимо смещение с 0 индекса при пагинации

// При первой рендере через onMounted отрисовываю все фото коллекции
onMounted(async () => {
  isLoading.value = true
  try {
    const category = categoryActive.value
      ? `category=${categoryActive.value}`
      : ''

    const url = `https://c6e8a13d4ed854c2.mokky.dev/photos?page=${page.value}&limit=3&${category}`
    // const url = `https://c6e8a13d4ed854c2.mokky.dev/photos?${category}`
    const result = await axios.get(url)

    // console.log(result)
    // console.log(result.data.items)

    collectionPhotos.value = result.data.items
    console.log('render1')
    console.log(collectionPhotos.value)
  } catch (error) {
    console.log(error)
  } finally {
    isLoading.value = false
  }
})

//Подписываюсь на реактивные переменные categoryActive и page
//Связанный запрос-фильтрация по категориям фото и одновременно пагинация с лимитом выдачи по 3 объекта на странице
watch([categoryActive, page], async () => {
  isLoading.value = true
  try {
    const category = categoryActive.value
      ? `category=${categoryActive.value}`
      : ''
    const url = `https://c6e8a13d4ed854c2.mokky.dev/photos?page=${page.value}&limit=3&${category}`
    // const url = `https://c6e8a13d4ed854c2.mokky.dev/photos?${category}`
    const result = await axios.get(url)

    //Присваиваю реактивной переменной данные приходящие от сервера
    //приходит сложный объект, нужно вытащить данные которые глубоко вложенные поэтому result.data.items
    collectionPhotos.value = result.data.items
    console.log('render2')
    // console.log(collectionPhotos.value)
  } catch (error) {
    console.log(error)
  } finally {
    isLoading.value = false
  }
})

// Фильтрую в инпуте поиска фото коллекции через запрос на сервер
const getFilteredResults = async () => {
  isLoading.value = true
  try {
    const title = searchValue.value
    // const url = `https://c6e8a13d4ed854c2.mokky.dev/photos?page=${page.value}&limit=3&name=*${title}`
    const url = `https://c6e8a13d4ed854c2.mokky.dev/photos?name=*${title}`
    const result = await axios.get(url)

    collectionPhotos.value = result.data
    console.log('render3')
  } catch (error) {
    console.log(error)
  } finally {
    isLoading.value = false
  }
}
//Задержка запроса на сервер через debounce из 'lodash/debounce'
const debouncedFilter = debounce(getFilteredResults, 1000) // 1 секунда
watch(() => searchValue.value, debouncedFilter)

// // Фильтрую в инпуте поиска фото коллекции по названию через computed(для сохранения  реактивности переменной)
// // Фильтрация без запроса на сервер
// const filteredPhotos = computed(() => {
//   return collectionPhotos.value.filter(obj =>
//     obj.name.toLowerCase().includes(searchValue.value.toLowerCase()),
//   )
// })
</script>

<template>
  <div class="App">
    <h1>Моя коллекция фотографий</h1>
    <!-- {{ console.log('render template') }} -->
      <!-- Реализация кнопок через список <li> categoryActive - это цифра которая уходит в запрос на сервер: -->
    <div class="wrap-buttons-input">
      <ul class="buttons-categories">
        <li
          v-for="(item, index) in categories"
          :key="item.name"
          :class="{ active: index === categoryActive }"
          @click="categoryActive = index"
        >
          {{ item.name }}
        </li>

        <!-- <li @click="getPhotos" class="active">Все</li>
        <li>Горы</li>
        <li>Море</li>
        <li>Архитектура</li>
        <li>Города</li> -->
      </ul>
      <input
        class="search-input"
        placeholder="Поиск по названию"
        v-model="searchValue"
      />
      <!-- {{ console.log(searchValue) }} -->
    </div>
    <div class="content" v-if="isLoading == false">
      <!-- Перебираю массив достаю из него пропсы и передаю их в дочерний компонент -->
      <!-- v-for="item in filteredPhotos" -->
      <CollectionPhoto
        v-for="item in collectionPhotos"
        :key="item.id"
        :photos="item.photos"
        :name="item.name"
      />
      <!-- <CollectionPhoto :photos="photos" /> -->
    </div>
    <h2 v-else>Идет загрузка ...</h2>
    <h2 v-if="collectionPhotos.length === 0">Ничего не найдено</h2>

    <ul  class="pagination">
      <li
        v-for="(item, index) in [1, 2, 3, 4, 5]"
        :key="index"
        :class="{ active: index + 1 === page }"
        @click="page = index + 1"
      >
        {{ item }}
      </li>
      <!-- <li>1</li>
      <li class="active">2</li>
      <li>3</li> -->
    </ul>
  </div>
</template>

<style lang="scss">
@import url('https://fonts.googleapis.com/css2?family=Merriweather:wght@400;700&display=swap');

body {
  margin: 0;
  font-family:
    system-ui,
    -apple-system,
    BlinkMacSystemFont,
    'Segoe UI',
    Roboto,
    Oxygen,
    Ubuntu,
    Cantarell,
    'Open Sans',
    'Helvetica Neue',
    sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  background-color: #d3eefd;
}

* {
  box-sizing: border-box;
  color: #007bff;
}

.App {
  padding: 50px;
  width: 100%;
  max-width: 1200px;
}

h1 {
  font-family: 'Merriweather', serif;
}

.buttons-categories {
  list-style: none;
  padding: 0;

  li {
    display: inline-block;
    padding: 12px 18px;
    background-color: #fff;
    border-radius: 10px;
    margin-right: 10px;
    cursor: pointer;
    font-weight: 500;
    font-size: 18px;
    border: 1px solid transparent;
    transition: all 0.15s ease-in-out;
    border: 1px solid rgba(19, 51, 255, 0.3);

    &:hover {
      border-color: #007bff;
    }

    &.active {
      background-color: #007bff;
      color: #fff;
    }
  }
}

.content {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  grid-column-gap: 30px;
  grid-row-gap: 30px;
  margin-top: 40px;

  @media (max-width: 1000px) {
    grid-template-columns: repeat(2, 1fr);
  }

  @media (max-width: 600px) {
    grid-template-columns: repeat(1, 1fr);
  }
}

.collection {
  max-width: 470px;
  background-color: #fff;
  padding: 30px;
  border-radius: 30px;
  box-shadow: 0 3px 0 rgba(19, 51, 255, 0.1);
  cursor: pointer;
  transition: all 0.15s ease-in-out;
  // border: 1px solid rgba(19, 51, 255, 0.3);

  h4 {
    margin: 0;
    margin-top: 15px;
    font-size: 20px;
    font-family: 'Merriweather', serif;
  }

  &:hover {
    box-shadow: 0 20px 30px rgba(19, 51, 255, 0.15);
    transform: translateY(-5px);
  }

  @media (max-width: 1000px) {
    width: 100%;
  }

  img {
    object-fit: cover;
  }

  &__big {
    height: 250px;
    width: 100%;
    border-radius: 20px;
    margin-bottom: 15px;
  }

  &__bottom {
    display: flex;
    justify-content: space-between;
  }

  &__mini {
    width: 31%;
    height: 80px;
    border-radius: 20px;
  }
}

.wrap-buttons-input {
  display: flex;
  align-items: center;
  flex-wrap: wrap;
}

.search-input {
  font-size: 16px;
  width: 250px;
  height: 50px;
  padding: 0 15px;
  border-radius: 10px;
  border: 1px solid rgba(19, 51, 255, 0.3);
  outline: none;
  transition: all 0.15s ease-in-out;

  &::placeholder {
    color: rgba(19, 51, 255, 0.4);
  }

  &:focus {
    border: 1px solid #007bff;
  }
}

.pagination {
  list-style: none;
  padding: 0;
  margin: 0;
  margin-top: 40px;

  li {
    display: inline-block;
    padding: 10px 17px;
    background-color: white;
    font-size: 18px;
    border-radius: 10px;
    margin-right: 15px;
    cursor: pointer;
    border: 1px solid rgba(19, 51, 255, 0.3);
    
    &:hover {
      border-color: #007bff;
    }

    &.active {
      background-color: #007bff;
      color: #fff;
    }
  }
}
</style>

<!-- Комментарии: -->
<!-- Во vue аналог setState (как вreact) реализуется путем присваивания (=) нового значения в реактивную переменную -->
<!-- const photos = [
  'https://images.unsplash.com/photo-1476574898132-040f50db0a01?q=80&w=3270&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D',
  'https://images.unsplash.com/photo-1476574898132-040f50db0a01?q=80&w=3270&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D',
  'https://images.unsplash.com/photo-1476574898132-040f50db0a01?q=80&w=3270&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D',
  'https://images.unsplash.com/photo-1476574898132-040f50db0a01?q=80&w=3270&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D',
]
const getPhotos = async () => {
  isLoading.value = true
  try {
    collectionPhotos.value = await axios.get(
      'https://c6e8a13d4ed854c2.mokky.dev/photos',
    )
    console.log(collectionPhotos.value.data)
  } catch (error) {
    console.log(error)
  } finally {
    isLoading.value = false
  }
}
getPhotos() -->

<!--  при преборе через v-for можно передавать несколько параметров (item, index) -->
<!--  правильная передача псевдоклассов зависящих от условия :class="{ active: index === categoryActive }" -->
<!--  Название классу давать только :class другие(className) работать не будут !!! -->
<!-- Фильтрую при нажатии кнопки @click="categoryActive = index"  почему присваиваю? это аналог setState:
В реакте было бы что-то подобное: {categories.map((obj, i) => (<li onClick={() = setCategoryActive(i)} </li>))}-->

<!-- В реактивную переменную collectionPhotos в template включено уже value т.е. collectionPhotos.value!!!
поэтому v-for="item in collectionPhotos.data" -->
