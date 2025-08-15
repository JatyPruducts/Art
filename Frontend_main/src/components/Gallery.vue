<template>
  <section ref="gallery" class="py-12 bg-gray-100">
    <div class="container mx-auto px-4">
      <h2 class="text-2xl font-bold mb-6">Наши картины</h2>
      
      <div v-if="paintingsToShow.length === 0" class="text-center py-10">
        <p class="text-gray-500">Картины не найдены. Попробуйте изменить параметры поиска.</p>
      </div>
      
      <div v-else>
        <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-6">
          <PaintingCard
            v-for="painting in paintingsToShow"
            :key="painting.id"
            :painting="painting"
            :is-admin="isAdmin"
            @details="$emit('details', painting)"
            @edit-painting="$emit('edit-painting', painting)"
            @delete-painting="$emit('delete-painting', painting)"
          />
        </div>
        
        <!-- Блок пагинации, который получает данные от родителя -->
        <div v-if="totalPages > 1" class="flex justify-center mt-8">
          <button
            @click="$emit('prev-page')"
            :disabled="currentPage === 1"
            class="px-4 py-2 mx-2 bg-gray-300 rounded disabled:opacity-50 hover:bg-gray-400 transition-colors"
          >
            Назад
          </button>
          <span class="px-4 py-2">Страница {{ currentPage }} из {{ totalPages }}</span>
          <button
            @click="$emit('next-page')"
            :disabled="currentPage === totalPages"
            class="px-4 py-2 mx-2 bg-gray-300 rounded disabled:opacity-50 hover:bg-gray-400 transition-colors"
          >
            Вперед
          </button>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import PaintingCard from './PaintingCard.vue';

export default {
  name: 'Gallery',
  components: { PaintingCard },
  props: {
    paintingsToShow: {
      type: Array,
      required: true,
    },
    isAdmin: {
      type: Boolean,
      default: false
    },
    currentPage: {
      type: Number,
      required: true
    },
    totalPages: {
      type: Number,
      required: true
    }
  },
  emits: ['details', 'edit-painting', 'delete-painting', 'prev-page', 'next-page'],
};
</script>