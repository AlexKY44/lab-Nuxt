<template>
  <div class="p-6">
    <UCard>
      <div class="flex justify-between items-center mb-4">
        <h1 class="text-2xl font-semibold">Products</h1>
        <UInput
            v-model="search"
            placeholder="Search products..."
            icon="i-heroicons-magnifying-glass-20-solid"
        />
      </div>

      <UTable
          :rows="paginatedRows"
          :columns="columns"
          :sort="sort"
          @update:sort="onSort"
      >
        <template #thumbnail-data="{ row }">
          <img :src="row.thumbnail" class="w-[100px] h-[100px] object-cover rounded" />
        </template>
        <template #rating-data="{ row }">
          <span
              :class="{
              'text-green-600': row.rating >= 4,
              'text-yellow-600': row.rating >= 3 && row.rating < 4,
              'text-red-600': row.rating < 3
            }"
          >
            {{ row.rating }}
          </span>
        </template>
      </UTable>

      <div class="flex justify-end mt-4">
        <UPagination
            v-model="page"
            :page-count="pageCount"
            :total="filteredRows.length"
        />
      </div>
    </UCard>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const products = ref([])
const search = ref('')
const sort = ref({ column: 'title', direction: 'asc' })
const page = ref(1)
const pageCount = 10

const columns = [
  { key: 'thumbnail', label: 'Image' },
  { key: 'title', label: 'Title', sortable: true },
  { key: 'description', label: 'Description' },
  { key: 'price', label: 'Price', sortable: true },
  { key: 'rating', label: 'Rating', sortable: true },
  { key: 'brand', label: 'Brand', sortable: true },
  { key: 'category', label: 'Category', sortable: true }
]

// Завантаження даних з API
onMounted(async () => {
  const res = await fetch('https://dummyjson.com/products?limit=100')
  const data = await res.json()
  products.value = data.products
})

// Пошук
const filteredRows = computed(() => {
  if (!search.value) return products.value

  return products.value.filter(product =>
      Object.values(product).some(val =>
          String(val).toLowerCase().includes(search.value.toLowerCase())
      )
  )
})

// Сортування
const sortedRows = computed(() => {
  if (!sort.value.column) return filteredRows.value

  return [...filteredRows.value].sort((a, b) => {
    const valA = a[sort.value.column]
    const valB = b[sort.value.column]

    if (valA < valB) return sort.value.direction === 'asc' ? -1 : 1
    if (valA > valB) return sort.value.direction === 'asc' ? 1 : -1
    return 0
  })
})

const paginatedRows = computed(() => {
  const start = (page.value - 1) * pageCount
  return sortedRows.value.slice(start, start + pageCount)
})

function onSort(updatedSort) {
  sort.value = updatedSort
}
</script>
