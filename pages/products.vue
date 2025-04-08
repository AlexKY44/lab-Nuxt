<script setup lang="ts">
import { ref, h, onMounted } from 'vue'
import { useHead } from '#imports'
import type { TableColumn } from '@nuxt/ui'

useHead({
  title: 'Список продуктів'
})

type Product = {
  id: number
  title: string
  description: string
  price: number
  rating: number
  brand: string
  category: string
  thumbnail: string
}

const data = ref<Product[]>([])
const loading = ref(true)
const globalFilter = ref('')

onMounted(async () => {
  try {
    const res = await fetch('https://dummyjson.com/products')
    const json = await res.json()
    data.value = json.products
  } catch (e) {
    console.error('Помилка при завантаженні продуктів:', e)
  } finally {
    loading.value = false
  }
})

const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'thumbnail',
    header: 'Фото',
    cell: ({ row }) =>
        h('img', {
          src: row.getValue('thumbnail'),
          alt: 'Thumbnail',
          width: 100,
          height: 100,
          class: 'object-cover rounded'
        })
  },
  {
    accessorKey: 'title',
    header: 'Назва'
  },
  {
    accessorKey: 'description',
    header: 'Опис'
  },
  {
    accessorKey: 'price',
    header: 'Ціна',
    cell: ({ row }) => `$${row.getValue('price')}`
  },
  {
    accessorKey: 'rating',
    header: 'Оцінка',
    cell: ({ row }) => {
      const rating = row.getValue('rating') as number

      return h('span', {
        class: [
          'font-semibold',
          rating < 4.5 ? 'text-red-500' : 'text-green-500'
        ]
      }, `${rating}/5`)
    }
  },
  {
    accessorKey: 'brand',
    header: 'Бренд'
  },
  {
    accessorKey: 'category',
    header: 'Категорія'
  }
]
</script>

<template>
  <div class="flex flex-col flex-1 w-full">
    <div class="flex px-4 py-3.5 border-b border-gray-200">
      <UInput v-model="globalFilter" class="max-w-sm" placeholder="Фільтрувати продукти..." />
    </div>

    <div class="p-4">
      <UTable
          v-if="!loading"
          :data="data"
          :columns="columns"
          v-model:global-filter="globalFilter"
          :sort="true"
          :paginate="true"
          :per-page="10"
      />
      <div v-else class="text-center py-10 text-gray-500">Завантаження продуктів...</div>
    </div>
  </div>
</template>
