<script setup lang="ts">
import { ref, computed, onMounted, watch } from 'vue'

// Оптимізація заголовка сторінки
useHead({
  title: 'Список продуктів'
})

// Стани для таблиці
const products = ref<
    {
      id: number
      title: string
      description: string
      price: number
      rating: number
      brand: string
      category: string
      thumbnail: string
    }[]
>([])
const currentPage = ref(1)
const rowsPerPage = ref(5)
const filterText = ref('')
const isLoading = ref(true)
const sortKey = ref('')
const sortDirection = ref<'asc' | 'desc'>('asc')

// Завантаження даних з API
const fetchData = async () => {
  isLoading.value = true
  try {
    const response = await fetch('https://dummyjson.com/products')
    const data = await response.json()
    products.value = data.products
  } catch (error) {
    console.error('Помилка завантаження даних:', error)
  } finally {
    isLoading.value = false
  }
}

onMounted(fetchData)

// Список із фільтром
const filteredData = computed(() =>
    products.value.filter((row) =>
        Object.values(row).some((value) =>
            String(value).toLowerCase().includes(filterText.value.toLowerCase())
        )
    )
)

// Список із сортуванням
const sortedData = computed(() =>
    filteredData.value.slice().sort((a, b) => {
      if (!sortKey.value) return 0
      const valueA = a[sortKey.value as keyof typeof a]
      const valueB = b[sortKey.value as keyof typeof b]
      if (valueA < valueB) return sortDirection.value === 'asc' ? -1 : 1
      if (valueA > valueB) return sortDirection.value === 'asc' ? 1 : -1
      return 0
    })
)

// Дані для поточної сторінки
const paginatedData = computed(() =>
    sortedData.value.slice(
        (currentPage.value - 1) * rowsPerPage.value,
        currentPage.value * rowsPerPage.value
    )
)

// Зміна напрямку сортування
const changeSort = (key: string) => {
  if (sortKey.value === key) {
    sortDirection.value = sortDirection.value === 'asc' ? 'desc' : 'asc'
  } else {
    sortKey.value = key
    sortDirection.value = 'asc'
  }
}

// Відстеження зміни рядка пошуку
watch(filterText, () => {
  currentPage.value = 1 // Перехід на першу сторінку
})
</script>

<template>
  <div class="container">
    <h1 class="title">Список продуктів</h1>

    <!-- Пошук -->
    <div class="search-wrap">
      <input
          type="text"
          v-model="filterText"
          placeholder="🔍 Пошук продуктів..."
          class="search-input"
      />
    </div>

    <!-- Таблиця -->
    <div class="table-container">
      <n-table v-if="!isLoading" class="custom-table">
        <thead>
        <tr>
          <th @click="changeSort('title')" class="sortable">
            Назва
            <span v-if="sortKey === 'title'">({{ sortDirection }})</span>
          </th>
          <th>Опис</th>
          <th @click="changeSort('price')" class="sortable">
            Ціна
            <span v-if="sortKey === 'price'">({{ sortDirection }})</span>
          </th>
          <th @click="changeSort('rating')" class="sortable">
            Оцінка
            <span v-if="sortKey === 'rating'">({{ sortDirection }})</span>
          </th>
          <th>Бренд</th>
          <th>Категорія</th>
          <th>Фото</th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="product in paginatedData" :key="product.id" class="table-row">
          <td>{{ product.title }}</td>
          <td>{{ product.description }}</td>
          <td>{{ product.price }} $</td>
          <td :class="{ 'high-rating': product.rating >= 4.5, 'low-rating': product.rating < 4.5 }">
            {{ product.rating }}
          </td>
          <td>{{ product.brand }}</td>
          <td>{{ product.category }}</td>
          <td>
            <img :src="product.thumbnail" alt="Фото" class="product-img" />
          </td>
        </tr>
        </tbody>
      </n-table>

      <!-- Завантаження -->
      <div v-else class="loading">Завантаження...</div>
    </div>

    <!-- Пагінація -->
    <div class="pagination">
      <button :disabled="currentPage === 1" @click="currentPage--" class="btn">
        Назад
      </button>
      <span>Сторінка {{ currentPage }} із {{ Math.ceil(filteredData.length / rowsPerPage) }}</span>
      <button
          :disabled="currentPage === Math.ceil(filteredData.length / rowsPerPage)"
          @click="currentPage++"
          class="btn"
      >
        Вперед
      </button>
    </div>
  </div>
</template>

<style scoped>
/* Загальний стиль контейнера */
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1.5rem;
}

/* Заголовок */
.title {
  font-family: 'Inter', sans-serif;
  font-size: 2rem;
  font-weight: 600;
  text-align: center;
  color: #1a202c;
  margin-bottom: 2rem;
}

/* Поле пошуку */
.search-wrap {
  margin-bottom: 1.5rem;
}

.search-input {
  width: 100%;
  padding: 0.5rem 1rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 0.5rem;
  outline: none;
  transition: border-color 0.3s;
}

.search-input:focus {
  border-color: #3182ce;
}

/* Таблиця */
.table-container {
  overflow-x: auto;
}

.custom-table thead {
  background-color: #f7fafc;
}

.custom-table th {
  padding: 0.75rem;
  text-align: left;
  font-weight: bold;
  color: #4a5568;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

.custom-table th.sortable:hover {
  background-color: #ebf8ff;
}

.custom-table tbody .table-row:hover {
  background-color: #f9fafb;
}

.custom-table td {
  padding: 0.75rem;
  color: #2d3748;
}

.high-rating {
  color: #38a169;
}

.low-rating {
  color: #e53e3e;
}

/* Фото */
.product-img {
  width: 64px;
  height: 64px;
  border-radius: 0.375rem;
  object-fit: cover;
}

/* Пагінація */
.pagination {
  margin-top: 1.5rem;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.btn {
  padding: 0.5rem 1rem;
  font-size: 1rem;
  border: none;
  border-radius: 0.375rem;
  background-color: #4299e1;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
}

.btn:hover:not(:disabled) {
  background-color: #3182ce;
  transform: translateY(-2px);
}

.btn:disabled {
  background-color: #e2e8f0;
  cursor: not-allowed;
}
</style>