<template>
  <section class="py-8 bg-white shadow-md">
    <div class="container mx-auto px-4">
      <h2 class="text-2xl font-bold mb-6 text-center md:text-left">Поиск картин</h2>
      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-x-6 gap-y-4 items-start">
        
        <!-- Колонка 1: Название и Теги -->
        <div class="space-y-4">
          <div>
            <label for="search-title" class="block text-sm font-medium text-gray-700 mb-1">По названию</label>
            <input type="text" id="search-title" placeholder="Введите название..."
                   v-model.trim="title"
                   @keyup.enter="emitSearch"
                   class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" />
          </div>
          
          <div>
            <h3 class="text-sm font-medium text-gray-700 mb-1">Теги:</h3>
            <div class="flex flex-wrap gap-2">
              <!-- Всегда видимые теги -->
              <button v-for="tag in visibleTags" :key="tag"
                      @click="toggleTag(tag)"
                      :class="[
                        'px-3 py-1 text-sm border rounded-full transition-colors duration-150 ease-in-out',
                        selectedTags.includes(tag) ? 'bg-blue-500 text-white border-blue-500 hover:bg-blue-600' : 'bg-gray-100 text-gray-700 border-gray-300 hover:bg-gray-200'
                      ]">
                {{ tag }}
              </button>
              <!-- Кнопка "Показать еще / Скрыть" -->
              <button v-if="hiddenTagsCount > 0"
                      @click="tagsExpanded = !tagsExpanded"
                      class="px-3 py-1 text-sm text-blue-600 hover:text-blue-800 rounded-full focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-offset-1">
                {{ tagsExpanded ? 'Скрыть' : `Еще ${hiddenTagsCount}...` }}
              </button>
            </div>
            <!-- Раскрываемые теги -->
            <transition
              enter-active-class="transition ease-out duration-200"
              enter-from-class="opacity-0 max-h-0"
              enter-to-class="opacity-100 max-h-96"
              leave-active-class="transition ease-in duration-150"
              leave-from-class="opacity-100 max-h-96"
              leave-to-class="opacity-0 max-h-0"
            >
              <div v-if="tagsExpanded && hiddenTagsCount > 0" class="mt-2 flex flex-wrap gap-2 overflow-hidden">
                <button v-for="tag in hiddenTags" :key="tag"
                        @click="toggleTag(tag)"
                        :class="[
                          'px-3 py-1 text-sm border rounded-full transition-colors duration-150 ease-in-out',
                          selectedTags.includes(tag) ? 'bg-blue-500 text-white border-blue-500 hover:bg-blue-600' : 'bg-gray-100 text-gray-700 border-gray-300 hover:bg-gray-200'
                        ]">
                  {{ tag }}
                </button>
              </div>
            </transition>
          </div>
        </div>

        <!-- Колонка 2: Размеры -->
        <div class="space-y-4">
          <div>
            <h3 class="text-sm font-medium text-gray-700 mb-1">Ширина (см):</h3>
            <div class="flex items-center gap-2">
              <input type="number" placeholder="От" min="0"
                     v-model.number="widthMin"
                     @keyup.enter="emitSearch"
                     class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" />
              <span class="text-gray-500">—</span>
              <input type="number" placeholder="До" min="0"
                     v-model.number="widthMax"
                     @keyup.enter="emitSearch"
                     class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" />
            </div>
          </div>
          <div>
            <h3 class="text-sm font-medium text-gray-700 mb-1">Высота (см):</h3>
            <div class="flex items-center gap-2">
              <input type="number" placeholder="От" min="0"
                     v-model.number="heightMin"
                     @keyup.enter="emitSearch"
                     class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" />
              <span class="text-gray-500">—</span>
              <input type="number" placeholder="До" min="0"
                     v-model.number="heightMax"
                     @keyup.enter="emitSearch"
                     class="w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" />
            </div>
          </div>
        </div>

        <!-- Колонка 3: Кнопка Поиска -->
        <div class="flex items-end pt-5 md:pt-0"> <!-- pt-5 для мобильных, чтобы кнопка была ниже полей -->
          <button @click="emitSearch"
                  class="w-full lg:w-auto lg:ml-auto px-6 py-2.5 bg-green-600 text-white font-medium rounded-md hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2 transition-colors">
            Найти
          </button>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
const VISIBLE_TAGS_COUNT = 5;

export default {
  name: 'SearchBar',
  props: {
    // НОВЫЙ PROP: принимаем готовый массив тегов
    allTags: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      title: '',
      selectedTags: [],
      widthMin: null,
      widthMax: null,
      heightMin: null,
      heightMax: null,
      tagsExpanded: false,
    };
  },
  computed: {
    // `allTags` теперь prop, а не computed.
    // Остальные computed зависят от него.
    visibleTags() {
      return this.allTags.slice(0, VISIBLE_TAGS_COUNT);
    },
    hiddenTags() {
      return this.allTags.slice(VISIBLE_TAGS_COUNT);
    },
    hiddenTagsCount() {
      return this.hiddenTags.length;
    }
  },
  methods: {
    toggleTag(tag) {
      const index = this.selectedTags.indexOf(tag);
      if (index > -1) {
        this.selectedTags.splice(index, 1);
      } else {
        this.selectedTags.push(tag);
      }
    },
    emitSearch() {
      const parseNumericInput = (val) => (val === null || val === '') ? null : Number(val);
      
      // Создаем объект только с непустыми значениями
      const searchCriteria = {};
      if (this.title) searchCriteria.title = this.title;
      if (this.selectedTags.length > 0) searchCriteria.tags = [...this.selectedTags];  // Отправляем как строку
      if (parseNumericInput(this.widthMin) !== null) searchCriteria.width_min = parseNumericInput(this.widthMin);
      if (parseNumericInput(this.widthMax) !== null) searchCriteria.width_max = parseNumericInput(this.widthMax);
      if (parseNumericInput(this.heightMin) !== null) searchCriteria.height_min = parseNumericInput(this.heightMin);
      if (parseNumericInput(this.heightMax) !== null) searchCriteria.height_max = parseNumericInput(this.heightMax);

      this.$emit('search', searchCriteria);
    }
  },
  watch: {
    allTags() {
      this.tagsExpanded = false;
    }
  }
};
</script>