<template>
  <transition
    enter-active-class="ease-out duration-300"
    enter-from-class="opacity-0"
    enter-to-class="opacity-100"
    leave-active-class="ease-in duration-200"
    leave-from-class="opacity-100"
    leave-to-class="opacity-0"
  >
    <div v-if="show" class="fixed inset-0 bg-gray-500 bg-opacity-75 transition-opacity flex justify-center items-center z-40 p-4" @click.self="closeModal">
      <transition
        enter-active-class="ease-out duration-300"
        enter-from-class="opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
        enter-to-class="opacity-100 translate-y-0 sm:scale-100"
        leave-active-class="ease-in duration-200"
        leave-from-class="opacity-100 translate-y-0 sm:scale-100"
        leave-to-class="opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
      >
        <div class="bg-white p-6 sm:p-8 rounded-lg shadow-xl w-full max-w-xl mx-4 my-8 overflow-y-auto max-h-[90vh]" role="dialog" aria-modal="true" aria-labelledby="modal-title">
          <div class="flex justify-between items-center mb-6">
            <h2 id="modal-title" class="text-xl sm:text-2xl font-semibold text-gray-900">{{ modalTitle }}</h2>
            <button @click="closeModal" class="text-gray-400 hover:text-gray-600 transition">
              <span class="sr-only">Закрыть</span>
              <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
            </button>
          </div>
          <form @submit.prevent="handleSubmit" novalidate>
            <!-- Все поля формы остаются без изменений -->
            <!-- Название картины -->
            <div class="mb-4">
              <label for="title" class="block text-sm font-medium text-gray-700 mb-1">Название картины <span class="text-red-500">*</span></label>
              <input type="text" id="title" v-model.trim="form.title" required class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" />
              <p v-if="errors.title" class="text-red-500 text-xs mt-1">{{ errors.title }}</p>
            </div>

            <!-- Размеры -->
            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 mb-4">
              <div>
                <label for="width" class="block text-sm font-medium text-gray-700 mb-1">Ширина (см) <span class="text-red-500">*</span></label>
                <input type="number" id="width" min="1" v-model.number="form.width" required class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" />
                <p v-if="errors.width" class="text-red-500 text-xs mt-1">{{ errors.width }}</p>
              </div>
              <div>
                <label for="height" class="block text-sm font-medium text-gray-700 mb-1">Высота (см) <span class="text-red-500">*</span></label>
                <input type="number" id="height" min="1" v-model.number="form.height" required class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" />
                <p v-if="errors.height" class="text-red-500 text-xs mt-1">{{ errors.height }}</p>
              </div>
            </div>

            <!-- Тэги -->
            <div class="mb-4">
              <label for="tags" class="block text-sm font-medium text-gray-700 mb-1">Тэги (через запятую) <span class="text-red-500">*</span></label>
              <input type="text" id="tags" v-model="form.tags" required placeholder="например, море, закат, абстракция" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" />
              <p v-if="errors.tags" class="text-red-500 text-xs mt-1">{{ errors.tags }}</p>
            </div>

            <!-- Описание -->
            <div class="mb-4">
              <label for="description" class="block text-sm font-medium text-gray-700 mb-1">Описание картины (опционально)</label>
              <textarea id="description" v-model.trim="form.description" rows="3" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"></textarea>
            </div>

            <!-- Загрузка фотографий -->
            <div class="mb-6">
              <label class="block text-sm font-medium text-gray-700 mb-1">Фотографии (до {{ MAX_FILES }} шт., макс. {{ MAX_FILE_SIZE_MB }}MB каждая)</label>
              <div v-if="isEditMode" class="text-xs text-gray-500 mb-2 p-2 bg-yellow-50 rounded-md">
                <b>Внимание:</b> Загрузка новых файлов полностью заменит существующие фотографии. Если вы не хотите менять изображения, просто оставьте это поле пустым.
              </div>
              <input type="file" ref="fileInput" multiple @change="handleFileUpload" accept="image/jpeg, image/png, image/gif, image/webp" class="block w-full text-sm text-gray-500 file:mr-4 file:py-2 file:px-4 file:rounded-md file:border-0 file:text-sm file:font-semibold file:bg-indigo-50 file:text-indigo-700 hover:file:bg-indigo-100 transition" />
              <p v-if="errors.images" class="text-red-500 text-xs mt-1">{{ errors.images }}</p>
              <!-- Превью -->
              <div v-if="imagePreviews.length > 0" class="mt-3 grid grid-cols-2 sm:grid-cols-3 md:grid-cols-4 lg:grid-cols-5 gap-3">
                <div v-for="(preview, index) in imagePreviews" :key="index" class="relative group">
                  <img :src="preview.url" :alt="'Превью ' + preview.name" class="w-full h-24 object-cover rounded-md border border-gray-200">
                  <button v-if="!preview.isExisting" type="button" @click="removeImage(index)" class="absolute top-1 right-1 bg-red-500 text-white rounded-full p-0.5 text-xs leading-none opacity-75 group-hover:opacity-100 transition-opacity">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-3 w-3" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M4.293 4.293a1 1 0 011.414 0L10 8.586l4.293-4.293a1 1 0 111.414 1.414L11.414 10l4.293 4.293a1 1 0 01-1.414 1.414L10 11.414l-4.293 4.293a1 1 0 01-1.414-1.414L8.586 10 4.293 5.707a1 1 0 010-1.414z" clip-rule="evenodd" /></svg>
                  </button>
                  <p class="text-xs text-gray-500 truncate mt-1" :title="preview.name">{{ preview.name }} <span v-if="preview.size">({{ (preview.size / 1024).toFixed(1) }}KB)</span></p>
                </div>
              </div>

            </div>

            <p v-if="formSubmitError" class="text-red-600 text-sm mb-4 bg-red-50 p-3 rounded-md">{{ formSubmitError }}</p>

            <div class="flex items-center justify-end space-x-3 pt-4 border-t border-gray-200">
              <button type="button" @click="closeModal" class="px-4 py-2 text-sm font-medium text-gray-700 bg-gray-100 hover:bg-gray-200 rounded-md focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition">Отмена</button>
              <button type="submit" :disabled="isSubmitting" class="px-4 py-2 text-sm font-medium text-white bg-indigo-600 hover:bg-indigo-700 rounded-md focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 disabled:opacity-50 transition">
                <span v-if="isSubmitting">Сохранение...</span>
                <span v-else>{{ submitButtonText }}</span>
              </button>
            </div>
          </form>
        </div>
      </transition>
    </div>
  </transition>
</template>

<script>
import axios from 'axios'

const MAX_FILES = 5;
const MAX_FILE_SIZE_MB = 2;

export default {
  name: 'AddPaintingModal',
  props: {
    show: {
      type: Boolean,
      required: true,
    },
    paintingToEdit: { // Данные для редактирования
      type: Object,
      default: null,
    },
  },
  data() {
    return {
      MAX_FILES,
      MAX_FILE_SIZE_MB,
      form: this.getInitialFormState(),
      errors: {},
      formSubmitError: '',
      isSubmitting: false,
      previewUrls: [], // Для хранения Object URLs для новых файлов
    };
  },
  computed: {
    isEditMode() {
      return !!this.paintingToEdit;
    },
    modalTitle() {
      return this.isEditMode ? 'Редактировать картину' : 'Добавить новую картину';
    },
    submitButtonText() {
      return this.isEditMode ? 'Сохранить изменения' : 'Добавить картину';
    },
    imagePreviews() {
            // Получаем базовый URL из переменных окружения
            const BASE_URL = import.meta.env.VITE_API_BASE_URL;

            // 1. Превью для СУЩЕСТВУЮЩИХ изображений (в режиме редактирования)
            const existingImages = (this.isEditMode && this.form.imageFiles.length === 0)
                // Используем правильное поле `photo_filenames` из `paintingToEdit`
                ? (this.paintingToEdit.photo_filenames || []).map(urlPath => ({
                    // Собираем ПОЛНЫЙ URL
                    url: `${BASE_URL}${urlPath}`, 
                    name: urlPath.split('/').pop(),
                    isExisting: true
                }))
                : [];

            // 2. Превью для НОВЫХ выбранных файлов (используя локальные Object URLs)
            const newImages = this.form.imageFiles.map((file, index) => ({
                url: this.previewUrls[index], // Это локальный временный URL
                name: file.name,
                size: file.size,
                isExisting: false
            }));

            // Если есть новые файлы, показываем только их. Иначе - существующие.
            return newImages.length > 0 ? newImages : existingImages;
        }
    },
    watch: {
        show(newVal) {
            if (newVal) {
                this.resetFormAndErrors();
                if (this.isEditMode) {
                    // 1. Добавляем console.log для проверки
                    console.log('Данные, пришедшие в модалку:', this.paintingToEdit);
                    
                    // Заполняем форму
                    this.form.title = this.paintingToEdit.title;
                    this.form.width = this.paintingToEdit.width;
                    this.form.height = this.paintingToEdit.height;
                    this.form.tags = this.paintingToEdit.tags.join(', ');
                    this.form.description = this.paintingToEdit.description;
                    
                    // 2. ИСПРАВЛЯЕМ ЭТУ СТРОКУ
                    // Было: this.form.id = this.paintingToEdit.id;
                    // Стало:
                    this.form.id = this.paintingToEdit.article_code; 
                }
            } else {
                this.revokePreviewUrls();
            }
        }
    },
  methods: {
    getInitialFormState() {
      return {
        id: null, 
        article_code: null, 
        title: '',
        width: null,
        height: null,
        tags: '',
        description: '',
        imageFiles: [],
      };
    },
    validateForm() {
      const newErrors = {};
      if (!this.form.title) newErrors.title = 'Название картины обязательно.';
      if (!this.form.width || this.form.width <= 0) newErrors.width = 'Ширина должна быть положительным числом.';
      if (!this.form.height || this.form.height <= 0) newErrors.height = 'Высота должна быть положительным числом.';
      if (!this.form.tags) newErrors.tags = 'Укажите хотя бы один тэг.';
      if (!this.isEditMode && this.form.imageFiles.length === 0) {
        newErrors.images = 'Загрузите хотя бы одну фотографию.';
      }

      this.errors = newErrors;
      return Object.keys(newErrors).length === 0;
    },
    handleFileUpload(event) {
        this.errors.images = ''; // Сбрасываем старую ошибку
        const newFiles = Array.from(event.target.files);

        if (newFiles.length > MAX_FILES) {
            this.errors.images = `Можно выбрать не более ${MAX_FILES} файлов.`;
            this.$refs.fileInput.value = ''; // Сбрасываем input
            return;
        }
        
        let currentFiles = [];
        let localErrorMessages = [];

        for (const file of newFiles) {
            if (file.size > MAX_FILE_SIZE_MB * 1024 * 1024) {
                localErrorMessages.push(`Файл "${file.name}" слишком большой (макс. ${MAX_FILE_SIZE_MB}MB).`);
                continue;
            }
            if (!['image/jpeg', 'image/png', 'image/gif', 'image/webp'].includes(file.type)) {
                localErrorMessages.push(`Файл "${file.name}" имеет неподдерживаемый формат.`);
                continue;
            }
            currentFiles.push(file);
        }

        // Записываем проверенные файлы в состояние формы
        this.form.imageFiles = currentFiles;
        this.updatePreviewUrls(); // Обновляем URL для превью

        if (localErrorMessages.length > 0) {
            this.errors.images = localErrorMessages.join(' ');
        }
        
        // Сбрасываем значение инпута, чтобы можно было выбрать те же файлы снова
        // ВАЖНО: не сбрасываем, если есть ошибки валидации файлов
        if (localErrorMessages.length === 0) {
            // эту строку можно убрать, если она вызывает проблемы, но обычно она полезна
            // this.$refs.fileInput.value = ''; 
        }
    },


    async handleSubmit() {
        this.formSubmitError = '';
        if (!this.validateForm()) {
            this.formSubmitError = 'Пожалуйста, исправьте ошибки в форме.';
            return;
        }
        this.isSubmitting = true;

        const formData = new FormData();

        // Добавляем все поля формы
        formData.append('title', this.form.title);
        formData.append('width', this.form.width);
        formData.append('height', this.form.height);
        formData.append('tags', this.form.tags); // Отправляем как строку
        formData.append('description', this.form.description);

        // Добавляем файлы. Ключ 'images' должен совпадать с тем, что в FastAPI (images: List[UploadFile])
        if (this.form.imageFiles.length > 0) {
            this.form.imageFiles.forEach(file => {
                formData.append('images', file);
            });
        }

        // --- ВАЖНО: АУТЕНТИФИКАЦИЯ ---
        const token = localStorage.getItem('access_token');
        if (!token) {
            this.formSubmitError = 'Ошибка аутентификации. Пожалуйста, войдите в систему заново.';
            this.isSubmitting = false;
            return;
        }

        const headers = {
            'Content-Type': 'multipart/form-data',
            'Authorization': `Bearer ${token}`
        };

        const BASE_API_URL = `${import.meta.env.VITE_API_BASE_URL}/api/paintings`; // <-- ИЗМЕНИТЕ, ЕСЛИ НУЖНО

        try {
                let response;
                if (this.isEditMode) {
                    // Используем ID для URL
                    response = await axios.put(`${BASE_API_URL}/${this.form.id}`, formData, { headers });
                } else {
                    response = await axios.post(BASE_API_URL, formData, { headers });
                }
                
                // Эмитим событие 'painting-saved' вместо 'add-painting' для ясности
                this.$emit('painting-saved', response.data);
                this.closeModal();

        } catch (error) {
            console.error('Ошибка при отправке формы:', error);
            // Показываем более осмысленную ошибку от FastAPI
            this.formSubmitError = error.response?.data?.detail || 'Произошла ошибка на сервере. Попробуйте позже.';
        } finally {
            this.isSubmitting = false;
        }
    },
    updatePreviewUrls() {
        this.revokePreviewUrls();
        this.previewUrls = this.form.imageFiles.map(file => URL.createObjectURL(file));
    },
    revokePreviewUrls() {
        this.previewUrls.forEach(url => URL.revokeObjectURL(url));
        this.previewUrls = [];
    },
    removeImage(index) {
        this.form.imageFiles.splice(index, 1);
        this.previewUrls.splice(index, 1); // Также удаляем URL
        this.errors.images = '';
    },
    resetFormAndErrors() {
      this.revokePreviewUrls();
      this.form = this.getInitialFormState();
      this.errors = {};
      this.formSubmitError = '';
      this.isSubmitting = false;
      if (this.$refs.fileInput) {
        this.$refs.fileInput.value = '';
      }
    },
    closeModal() {
      this.$emit('close');
    },
  },
  beforeUnmount() {
    this.revokePreviewUrls();
  }
};
</script>