<script setup lang="ts">
import { ref, computed } from 'vue';
import { useAsyncData } from '#app';
import { UiTable } from '@nuxt/ui';

useHead({
  title: 'Список продуктів'
});

// Доступні дані
const { data: productsData, pending, error } = useAsyncData('products', () =>
    $fetch('https://dummyjson.com/products')
);

// Стан таблиці
const searchQuery = ref('');
const currentPage = ref(1);
const itemsPerPage = 10;

// Список продуктів
const allProducts = computed(() =>
    productsData.value?.products.map((product) => ({
      title: product.title,
      description: product.description,
      price: product.price,
      rating: product.rating,
      brand: product.brand,
      category: product.category,
      thumbnail: product.thumbnail
    })) || []
);

// Фільтрація по пошуковому запиту
const filteredProducts = computed(() => {
  if (!searchQuery.value) {
    return allProducts.value;
  }
  return allProducts.value.filter((product) =>
      Object.values(product).some((value) =>
          String(value).toLowerCase().includes(searchQuery.value.toLowerCase())
      )
  );
});

// Сортування даних
const sortBy = ref('');
const sortOrder = ref('asc');

const sortedProducts = computed(() => {
  if (!sortBy.value) {
    return filteredProducts.value;
  }
  return [...filteredProducts.value].sort((a, b) => {
    const valA = a[sortBy.value];
    const valB = b[sortBy.value];

    let comparison = 0;
    if (valA > valB) {
      comparison = 1;
    } else if (valA < valB) {
      comparison = -1;
    }
    return sortOrder.value === 'asc' ? comparison : -comparison;
  });
});

// Пагінація
const paginatedProducts = computed(() => {
  const startIdx = (currentPage.value - 1) * itemsPerPage;
  return sortedProducts.value.slice(startIdx, startIdx + itemsPerPage);
});

const totalPages = computed(() =>
    Math.ceil(sortedProducts.value.length / itemsPerPage)
);

const setSort = (key) => {
  if (sortBy.value === key) {
    sortOrder.value = sortOrder.value === 'asc' ? 'desc' : 'asc';
  } else {
    sortBy.value = key;
    sortOrder.value = 'asc';
  }
};
</script>

<template>
  <div>
    <h1>Список продуктів</h1>

    <!-- Поле для пошуку -->
    <input
        v-model="searchQuery"
        placeholder="Пошук продуктів"
        class="search-input"
    />

    <!-- Таблиця -->
    <UiTable :items="paginatedProducts">
      <template #default="{ item }">
        <tr>
          <td>{{ item.title }}</td>
          <td>{{ item.description }}</td>
          <td>{{ item.price }}</td>
          <td>
            <span
                :style="{ color: item.rating < 4.5 ? 'red' : 'green' }"
            >
              {{ item.rating }}
            </span>
          </td>
          <td>{{ item.brand }}</td>
          <td>{{ item.category }}</td>
          <td>
            <img
                :src="item.thumbnail"
                alt="Фото"
                class="thumbnail"
            />
          </td>
        </tr>
      </template>
    </UiTable>

    <!-- Пагінація -->
    <div class="pagination">
      <button
          @click="currentPage -= 1"
          :disabled="currentPage === 1"
      >
        Попередня
      </button>

      <span>Сторінка {{ currentPage }} з {{ totalPages }}</span>

      <button
          @click="currentPage += 1"
          :disabled="currentPage === totalPages"
      >
        Наступна
      </button>
    </div>
  </div>
</template>

<style scoped>
.thumbnail {
  width: 100px;
  height: 100px;
  object-fit: cover;
}
.search-input {
  padding: 8px;
  margin-bottom: 16px;
  width: 100%;
  max-width: 400px;
  border-radius: 4px;
  border: 1px solid #ccc;
}
.pagination {
  margin-top: 16px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}
</style>