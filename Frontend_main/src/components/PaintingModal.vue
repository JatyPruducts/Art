<!-- src/components/PaintingModal.vue -> <template> -->

<template>
  <transition
    enter-active-class="transition-opacity ease-out duration-300"
    enter-from-class="opacity-0"
    enter-to-class="opacity-100"
    leave-active-class="transition-opacity ease-in duration-200"
    leave-from-class="opacity-100"
    leave-to-class="opacity-0"
  >
    <div class="fixed inset-0 bg-gray-800 bg-opacity-75 flex items-center justify-center z-50 p-4" @click="$emit('close')">
      <transition
        enter-active-class="transition ease-out duration-300"
        enter-from-class="opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
        enter-to-class="opacity-100 translate-y-0 sm:scale-100"
        leave-active-class="transition ease-in duration-200"
        leave-from-class="opacity-100 translate-y-0 sm:scale-100"
        leave-to-class="opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
      >
        <div v-if="painting" class="bg-white rounded-xl shadow-xl max-w-3xl w-full mx-auto overflow-hidden relative" @click.stop>
          
          <button @click="$emit('close')" class="absolute top-3 right-3 text-gray-400 hover:text-gray-600 p-1 rounded-full focus:outline-none focus:ring-2 focus:ring-gray-400 z-30 bg-white bg-opacity-50 hover:bg-opacity-75">
            <span class="sr-only">Закрыть</span>
            <svg class="h-6 w-6" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" aria-hidden="true">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" />
            </svg>
          </button>

          <div class="flex flex-col md:flex-row max-h-[90vh]">
            <!-- Левая часть: ВОССТАНАВЛИВАЕМ ВСЮ ИНФОРМАЦИЮ -->
            <div class="md:w-1/2 p-5 sm:p-6 overflow-y-auto">
              <h2 :id="'modal-title-' + painting.id" class="text-xl sm:text-2xl font-bold mb-3 text-gray-900">{{ painting.title }}</h2>
              
              <p class="text-sm text-gray-500 mb-1">Размер: <span class="font-medium text-gray-700">{{ painting.width }}×{{ painting.height }} см</span></p>
              <p class="text-sm text-gray-500 mb-3">Артикул: <span class="font-medium text-gray-700">{{ painting.article_code }}</span></p>
              
              <div v-if="painting.tags && painting.tags.length > 0" class="mb-4">
                <h4 class="text-xs text-gray-500 uppercase font-semibold mb-1.5">Тэги:</h4>
                <div class="flex flex-wrap gap-1.5">
                  <span v-for="tag in painting.tags" :key="tag"
                        class="inline-block bg-indigo-100 text-indigo-700 text-xs font-semibold px-2 py-1 rounded-full">
                    {{ tag }}
                  </span>
                </div>
              </div>
              
              <div v-if="painting.description" class="mb-6">
                <h4 class="text-xs text-gray-500 uppercase font-semibold mb-1.5">Описание:</h4>
                <p class="text-gray-700 text-sm leading-relaxed whitespace-pre-line">{{ painting.description }}</p>
              </div>
              
              <button @click="$emit('callback', painting.article_code)"
                      class="w-full bg-blue-600 text-white px-4 py-2.5 rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-2 transition-colors text-sm font-medium mt-auto">
                Свяжитесь со мной
              </button>
            </div>

            <!-- Правая часть: Карусель изображений -->
            <div class="md:w-1/2 bg-gray-100 flex items-center justify-center p-2 md:p-4 relative min-h-[250px] md:min-h-0">
              <div v-if="painting.photo_filenames && painting.photo_filenames.length > 0" class="w-full h-full relative">
                <img v-show="!imageErrors[idx]" 
                  :src="getImageUrl(painting.photo_filenames[idx])" 
                  :alt="painting.title + ' - изображение ' + (idx + 1)"
                  class="w-full h-full object-contain" 
                  @error="handleCarouselImageError" />
              
                <div v-show="imageErrors[idx]" class="w-full h-full flex items-center justify-center bg-gray-200">
                  <!-- Плейсхолдер ошибки -->
                </div>
                
                <button v-if="painting.photo_filenames.length > 1" @click.stop="prev" class="carousel-button left-1 sm:left-2">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M15 19l-7-7 7-7" />
                  </svg>
                </button>
                <button v-if="painting.photo_filenames.length > 1" @click.stop="next" class="carousel-button right-1 sm:right-2">
                  <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M9 5l7 7-7 7" />
                  </svg>
                </button>
                
                <div v-if="painting.photo_filenames.length > 1" class="absolute bottom-2 left-1/2 transform -translate-x-1/2 flex space-x-1.5 z-10">
                    <button v-for="(img, i) in painting.photo_filenames" :key="i" @click.stop="idx = i"
                            :class="['w-2 h-2 rounded-full', idx === i ? 'bg-blue-500 ring-2 ring-blue-300' : 'bg-gray-400 hover:bg-gray-500', 'transition-all']"
                            :aria-label="'Перейти к изображению ' + (i + 1)"></button>
                </div>
              </div>
              <div v-else class="text-gray-400 p-8 text-center text-sm">Нет изображений для этой картины.</div>
            </div>
          </div>
        </div>
      </transition>
    </div>
  </transition>
</template>

<script>
export default {
  name: 'PaintingModal',
  props: {
    painting: { // Может быть null, пока модальное окно не открыто
      type: Object,
      default: null
    }
  },
  data() {
    return { 
      idx: 0 ,
      imageErrors: {}
    };
  },
  watch: {
    painting(newPainting, oldPainting) {
      this.imageErrors = {};
      if (newPainting && (!oldPainting || newPainting.id !== oldPainting.id)) {
        this.idx = 0;
      }
    }
  },
  methods: {
    next() { 
      if (this.painting && this.painting.photo_filenames && this.painting.photo_filenames.length > 0) {
        this.idx = (this.idx + 1) % this.painting.photo_filenames.length; 
      }
    },
    prev() { 
      if (this.painting && this.painting.photo_filenames && this.painting.photo_filenames.length > 0) {
        this.idx = (this.idx - 1 + this.painting.photo_filenames.length) % this.painting.photo_filenames.length; 
      }
    },
    handleCarouselImageError(event) {
      console.warn(`Ошибка загрузки изображения: ${event.target.src}`);
      this.imageErrors = {
        ...this.imageErrors,
        [this.idx]: true
      };
      event.target.onerror = null;
    },
    handleEscKey(event) {
      if (event.key === 'Escape') {
        this.$emit('close');
      }
    },
    getImageUrl(path) {
            if (!path) return ''; // Защита от ошибок
            const BASE_URL = import.meta.env.VITE_API_BASE_URL;
            return `${BASE_URL}${path}`;
        },
  },
  mounted() {
    window.addEventListener('keydown', this.handleEscKey);
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleEscKey);
  }
};
</script>

<style scoped>
.carousel-button {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background-color: rgba(0, 0, 0, 0.25);
  color: white;
  border: none;
  border-radius: 50%;
  cursor: pointer;
  z-index: 10;
  transition: background-color 0.2s;
  
  /* Железное центрирование с Flexbox */
  display: inline-flex;
  align-items: center;
  justify-content: center;

  /* Явные размеры и сброс лишних стилей */
  width: 2.5rem;  /* 40px */
  height: 2.5rem; /* 40px */
  padding: 0;
  line-height: 1; /* Убираем лишнюю высоту строки */
  font-size: 1.5rem; /* Размер самой стрелки */
}
.carousel-button:hover {
  background-color: rgba(0, 0, 0, 0.5);
}
.carousel-button:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
.whitespace-pre-line {
  white-space: pre-line;
}
.overflow-y-auto::-webkit-scrollbar {
  width: 6px;
}
.overflow-y-auto::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 10px;
}
.overflow-y-auto::-webkit-scrollbar-thumb {
  background: #c1c1c1;
  border-radius: 10px;
}
.overflow-y-auto::-webkit-scrollbar-thumb:hover {
  background: #a1a1a1;
}
</style>