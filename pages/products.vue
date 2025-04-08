<script setup lang="ts">
import { ref, h, onMounted } from 'vue'
import { resolveComponent } from 'vue'
import type { TableColumn } from '@nuxt/ui'

const UBadge = resolveComponent('UBadge')

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

// API завантаження
const data = ref<Product[]>([])
const loading = ref(true)

onMounted(async () => {
  try {
    const res = await fetch('https://dummyjson.com/products')
    const json = await res.json()
    data.value = json.products
  } catch (e) {
    console.error('Failed to load products:', e)
  } finally {
    loading.value = false
  }
})

const columns: TableColumn<Product>[] = [
  {
    accessorKey: 'thumbnail',
    header: 'Image',
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
    header: 'Title'
  },
  {
    accessorKey: 'description',
    header: 'Description'
  },
  {
    accessorKey: 'price',
    header: 'Price',
    cell: ({ row }) => `$${row.getValue('price')}`
  },
  {
    accessorKey: 'rating',
    header: 'Rating',
    cell: ({ row }) => {
      const rating = row.getValue('rating') as number
      let color = 'success'
      if (rating < 2) color = 'error'
      else if (rating < 4) color = 'warning'

      return h(UBadge, { color, variant: 'subtle' }, () => `${rating}/5`)
    }
  },
  {
    accessorKey: 'brand',
    header: 'Brand'
  },
  {
    accessorKey: 'category',
    header: 'Category'
  }
]

const globalFilter = ref('')
</script>

<template>
  <div class="flex flex-col flex-1 w-full">
    <div class="flex px-4 py-3.5 border-b border-(--ui-border-accented)">
      <UInput v-model="globalFilter" class="max-w-sm" placeholder="Filter products..." />
    </div>

    <div class="p-4">
      <UTable
          v-if="!loading"
          :data="data"
          :columns="columns"
          v-model:global-filter="globalFilter"
      />
      <div v-else class="text-center py-10">Loading products...</div>
    </div>
  </div>
</template>
