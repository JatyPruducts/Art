<template>
  <!-- Этот div размывается при открытии модальных окон -->
  <div :class="{ 'blur-sm pointer-events-none': selected || showPaintingModal || showAnalyticsModal }">
    <Header
      :is-admin="isAdmin"
      @admin-login="handleAdminLogin"
      @admin-logout="handleAdminLogout"
      @open-add-painting-modal="openAddPaintingModalHandler"
      @open-analytics-modal="openAnalyticsModalHandler"
    />
    <HeroSlider />
    <ArtistIntro />
    <!-- SearchBar получает ВСЕ картины для построения списка тегов -->
    <SearchBar :all-tags="allTags" @search="handleSearch" />
    <!-- Gallery получает уже отфильтрованные и нарезанные на страницы данные -->
    <Gallery
      :paintingsToShow="currentPaintings"
      :is-admin="isAdmin"
      :currentPage="currentPage"
      :totalPages="totalPages"
      @details="openModal"
      @edit-painting="handleEdit"
      @delete-painting="handleDelete"
      @prev-page="prevPage"
      @next-page="nextPage"
    />
    <CallbackForm
      ref="callbackForm"
      :preArt="callbackArt"
      @form-submitted="handleFormSubmission"
    />
    <Footer />
  </div>

  <!-- Модальные окна находятся СНАРУЖИ размываемого div'а -->
  <PaintingModal v-if="selected" :painting="selected" @close="closeModal" @callback="triggerCallback" />
  <AddPaintingModal :show="showPaintingModal" :painting-to-edit="paintingToEdit" @close="closePaintingModal" @painting-saved="handleSavePainting" />
  <AnalyticsModal
    :show="showAnalyticsModal"
    :visits-data="analyticsData.visits"
    :submissions-data="analyticsData.submissions"
    @close="closeAnalyticsModalHandler"
  />
</template>

<script>
import axios from 'axios';
import Header from './components/Header.vue';
import HeroSlider from './components/HeroSlider.vue';
import ArtistIntro from './components/ArtistIntro.vue';
import SearchBar from './components/SearchBar.vue';
import Gallery from './components/Gallery.vue';
import PaintingModal from './components/PaintingModal.vue';
import CallbackForm from './components/CallbackForm.vue';
import Footer from './components/Footer.vue';
import AddPaintingModal from './components/AddPaintingModal.vue';
import AnalyticsModal from './components/AnalyticsModal.vue';

const getTodayDateString = () => {
  const today = new Date();
  const year = today.getFullYear();
  const month = String(today.getMonth() + 1).padStart(2, '0');
  const day = String(today.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
};

export default {
  name: 'App',
  components: {
    Header, HeroSlider, ArtistIntro, SearchBar, Gallery,
    PaintingModal, CallbackForm, Footer, AddPaintingModal, AnalyticsModal
  },
  data() {
    return {
      // Админка и модальные окна
      isAdmin: false,
      showPaintingModal: false,
      paintingToEdit: null,
      selected: null,
      callbackArt: null,
      
      // НОВАЯ СТРУКТУРА ДАННЫХ
      allTags: [], // Здесь будут храниться все теги для SearchBar
      currentPaintings: [], // Картины ТОЛЬКО для текущей страницы
      
      // Параметры для запросов к API
      currentPage: 1,
      itemsPerPage: 12,
      totalPages: 1,
      currentSearchParams: {}, // Текущие активные фильтры

      // Аналитика
      showAnalyticsModal: false,
      analyticsData: { visits: {}, submissions: {} },
    };
  },
  created() {
    // Проверяем токен при загрузке
    if (localStorage.getItem('access_token')) {
        this.isAdmin = true;
    }
    // Запускаем первоначальную загрузку данных
    this.initializePage();
    this.loadAnalyticsData();
    this.trackVisit();
  },
  methods: {
    // --- НОВАЯ ЛОГИКА ЗАГРУЗКИ И ФИЛЬТРАЦИИ ---
    async initializePage() {
      // 1. Загружаем все теги один раз при старте
      await this.fetchAllTags();
      // 2. Загружаем первую страницу картин
      await this.fetchPaginatedPaintings();
    },

    async fetchAllTags() {
        try {
            const response = await axios.get(`${import.meta.env.VITE_API_BASE_URL}/api/paintings/tags/all`);
            this.allTags = response.data;
        } catch (error) {
            console.error("Ошибка при загрузке тегов:", error);
            alert("Не удалось загрузить список тегов.");
        }
    },

    async fetchPaginatedPaintings() {
      // Базовый URL для запросов
      const baseUrl = `${import.meta.env.VITE_API_BASE_URL}/api/paintings`;
      const countUrl = `${import.meta.env.VITE_API_BASE_URL}/api/paintings/count`;

      // Собираем параметры вручную, чтобы tags передавались как ?tags=tag1&tags=tag2
      const params = new URLSearchParams();
      params.append('skip', (this.currentPage - 1) * this.itemsPerPage);
      params.append('limit', this.itemsPerPage);

      // Добавляем текст поиска
      if (this.currentSearchParams.title) {
        params.append('title', this.currentSearchParams.title);
      }

      // Добавляем теги по одному
      if (Array.isArray(this.currentSearchParams.tags)) {
        this.currentSearchParams.tags.forEach(tag => {
          params.append('tags', tag);
        });
      }

      // Добавляем размеры
      if (this.currentSearchParams.width_min != null) {
        params.append('width_min', this.currentSearchParams.width_min);
      }
      if (this.currentSearchParams.width_max != null) {
        params.append('width_max', this.currentSearchParams.width_max);
      }
      if (this.currentSearchParams.height_min != null) {
        params.append('height_min', this.currentSearchParams.height_min);
      }
      if (this.currentSearchParams.height_max != null) {
        params.append('height_max', this.currentSearchParams.height_max);
      }

      try {
        // Запрос за списком картин
        const paintingsResponse = await axios.get(`${baseUrl}?${params.toString()}`);
        this.currentPaintings = paintingsResponse.data;

        // Запрос за общим количеством страниц
        const totalCountResponse = await axios.get(`${countUrl}?${params.toString()}`);
        this.totalPages = Math.ceil(totalCountResponse.data.total / this.itemsPerPage);

      } catch (error) {
        console.error("Ошибка при загрузке картин:", error);
        alert("Не удалось загрузить страницу с картинами.");
      }
    },

    async handleSearch(searchCriteria) {
      // Сохраняем параметры поиска и сбрасываем на 1-ю страницу
      this.currentSearchParams = searchCriteria;
      this.currentPage = 1;
      // Выполняем новый поиск
      await this.fetchPaginatedPaintings();
    },

    // --- Пагинация ---
    async prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
        await this.fetchPaginatedPaintings();
      }
    },
    async nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
        await this.fetchPaginatedPaintings();
      }
    },

    // --- CRUD и Модальные окна (логика немного меняется) ---
    async handleSavePainting() {
      this.closePaintingModal();
      // Перезагружаем теги (на случай, если добавили новый) и текущую страницу
      await this.initializePage();
      alert('Данные успешно сохранены!');
    },
    
    async handleDelete(paintingToDelete) {
      if (!window.confirm(`Вы уверены, что хотите удалить картину "${paintingToDelete.title}" (ID: ${paintingToDelete.id})?`)) return;
      try {
        const token = localStorage.getItem('access_token');
        const realId = paintingToDelete.article_code;

        console.log('Удаляем по article_code =', realId);
        await axios.delete(`${import.meta.env.VITE_API_BASE_URL}/api/paintings/${realId}`, {
            headers: { 'Authorization': `Bearer ${token}` }
        });
        alert(`Картина "${paintingToDelete.title}" успешно удалена.`);
        // Просто перезагружаем текущую страницу
        await this.fetchPaginatedPaintings();
      } catch (error) {
        console.error('Ошибка при удалении картины:', error);
        alert(error.response?.data?.detail || 'Ошибка на сервере при удалении картины');
      }
    },

    // Остальные методы остаются практически без изменений
    openModal(painting) { this.selected = painting; },
    closeModal() { this.selected = null; },
    handleEdit(painting) { this.paintingToEdit = painting; this.showPaintingModal = true; },
    openAddPaintingModalHandler() { this.paintingToEdit = null; this.showPaintingModal = true; },
    closePaintingModal() { this.paintingToEdit = null; this.showPaintingModal = false; },
    handleAdminLogin() { this.isAdmin = true; },
    handleAdminLogout() { this.isAdmin = false; localStorage.removeItem('access_token'); },
    triggerCallback(art) {
        this.callbackArt = art;
        this.closeModal();
        this.$nextTick(() => { this.$refs.callbackForm.$el.scrollIntoView({ behavior: 'smooth' }); });
    },

    // --- Аналитика ---
    loadAnalyticsData() {
      try {
        const visits = JSON.parse(localStorage.getItem('galleryVisitsByDate') || '{}');
        const submissions = JSON.parse(localStorage.getItem('gallerySubmissionsByDate') || '{}');
        this.analyticsData = { visits, submissions };
      } catch (e) { console.error("Failed to parse analytics data from localStorage", e); }
    },
    trackVisit() {
      const today = getTodayDateString();
      const currentVisits = this.analyticsData.visits[today] || 0;
      this.analyticsData.visits[today] = currentVisits + 1;
      localStorage.setItem('galleryVisitsByDate', JSON.stringify(this.analyticsData.visits));
    },
    handleFormSubmission() {
      const today = getTodayDateString();
      const currentSubmissions = this.analyticsData.submissions[today] || 0;
      this.analyticsData.submissions[today] = currentSubmissions + 1;
      localStorage.setItem('gallerySubmissionsByDate', JSON.stringify(this.analyticsData.submissions));
    },
    openAnalyticsModalHandler() {
      this.loadAnalyticsData();
      this.showAnalyticsModal = true;
    },
    closeAnalyticsModalHandler() { this.showAnalyticsModal = false; },
  }
};
</script>

<style>
/* 
  Класс blur-sm из Tailwind уже достаточно хорош.
  pointer-events-none нужен, чтобы предотвратить взаимодействие с фоном.
*/
.pointer-events-none {
  pointer-events: none;
}
</style>