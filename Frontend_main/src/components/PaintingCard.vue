<template>
  <div class="bg-white rounded-lg shadow-lg overflow-hidden flex flex-col transition-all duration-300 hover:shadow-xl relative group">
    <!-- Блок для изображения -->
    <div class="w-full h-56 bg-gray-100">
        <img
            v-if="imageUrl && !imageLoadError"
            :src="imageUrl" 
            :alt="painting.title"
            class="w-full h-full object-cover"
            @error="handleImageError"
        />
      <!-- Если проблемы с загрузкой (Отсутствует в папке по имеющемуся названию) -->
      <div v-if="imageLoadError" class="w-full h-full bg-gray-200 flex items-center justify-center text-gray-500 text-sm p-4 text-center">
        <div>
          <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8 mx-auto text-gray-400" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16l4.586-4.586a2 2 0 012.828 0L16 16m-2-2l1.586-1.586a2 2 0 012.828 0L20 14m-6-6h.01M6 20h12a2 2 0 002-2V6a2 2 0 00-2-2H6a2 2 0 00-2 2v12a2 2 0 002 2z" />
          </svg>
          <p class="mt-1">Фотография не загружена</p>
        </div>
      </div>
      <!-- Если изображение отсутствует -->
      <div v-else-if="!painting.photo_filenames || painting.photo_filenames.length === 0" 
           class="w-full h-full bg-gray-200 flex items-center justify-center text-gray-500 text-sm p-4 text-center">
        Фотография отсутствует
      </div>
    </div>

    <!-- Контейнер для текстового содержимого и кнопок -->
    <div class="p-4 sm:p-5 flex flex-col flex-grow">
      <h3 class="text-base sm:text-lg font-semibold mb-1 truncate" :title="painting.title">
        {{ painting.title || 'Без названия' }}
      </h3>
      <p class="text-gray-600 text-xs sm:text-sm mt-1">
        Артикул: {{ painting.article_code || 'N/A' }}
      </p>

      <!-- Блок с кнопками (основная и админские) -->
      <div class="mt-auto pt-3 flex items-center justify-between">
        <button @click="$emit('details', painting)"
                class="bg-blue-500 text-white px-3 sm:px-4 py-1.5 sm:py-2 rounded-md text-sm font-medium hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 transition-colors">
          Подробнее
        </button>

        <!-- Админские кнопки -->
        <div v-if="isAdmin" class="flex items-center space-x-2">
          <button @click.stop="$emit('edit-painting', painting)" title="Редактировать" class="p-2 text-gray-500 hover:text-blue-600 hover:bg-blue-100 rounded-full transition-colors">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M11 5H6a2 2 0 00-2 2v11a2 2 0 002 2h11a2 2 0 002-2v-5m-1.414-9.414a2 2 0 112.828 2.828L11.828 15H9v-2.828l8.586-8.586z" />
            </svg>
          </button>
          <button @click.stop="$emit('delete-painting', painting)" title="Удалить" class="p-2 text-gray-500 hover:text-red-600 hover:bg-red-100 rounded-full transition-colors">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
              <path stroke-linecap="round" stroke-linejoin="round" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
            </svg>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PaintingCard',
  props: {
    painting: {
      type: Object,
      required: true
    },
    isAdmin: { // Новый prop
      type: Boolean,
      default: false
    }
  },
  data(){
    return {
      imageLoadError: false
    }
  },
  computed: {
        imageUrl() {
            if (this.painting.photo_filenames && this.painting.photo_filenames.length > 0) {
                const BASE_URL = import.meta.env.VITE_API_BASE_URL;
                // Собираем ПОЛНЫЙ URL из базового URL бэкенда и пути к файлу
                return `${BASE_URL}${this.painting.photo_filenames[0]}`;
            }
            return null; // или URL для плейсхолдера по умолчанию
        }
    },
  methods: {
     handleImageError(event) {
      console.warn(`Ошибка загрузки изображения для карточки: ${event.target.src}. Артикул: ${this.painting.article_code}`);
      this.imageLoadError = true; 
      
      event.target.onerror = null;
    },
  },
  watch: {
    painting() {
      this.imageLoadError = false;
    }
  }
};
</script>