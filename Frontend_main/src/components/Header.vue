<template>
  <header class="bg-gray-800 text-white py-6 shadow-md">
    <div class="container mx-auto px-4 flex justify-between items-center">
      <div>
        <h1 class="text-2xl sm:text-3xl font-bold">Галерея картин Сабанова Степана Леонтьевича</h1>
        <p class="mt-1 sm:mt-2 text-sm sm:text-base text-gray-300">Уникальные произведения искусства для вашего дома</p>
      </div>

      <div class="relative">
        <button
          @click="toggleMenu"
          class="bg-blue-600 hover:bg-blue-700 text-white font-semibold py-2 px-4 rounded focus:outline-none focus:ring-2 focus:ring-blue-400 focus:ring-opacity-50"
          aria-haspopup="true"
          :aria-expanded="isMenuOpen.toString()"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 sm:hidden" viewBox="0 0 20 20" fill="currentColor">
            <path fill-rule="evenodd" d="M3 5a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zM3 10a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zM3 15a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1z" clip-rule="evenodd" />
          </svg>
          <span class="hidden sm:inline">Меню</span>
          <svg class="hidden sm:inline-block w-4 h-4 ml-1 fill-current" viewBox="0 0 20 20">
            <path d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" fill-rule="evenodd"></path>
          </svg>
        </button>

        <transition
          enter-active-class="transition ease-out duration-100"
          enter-from-class="transform opacity-0 scale-95"
          enter-to-class="transform opacity-100 scale-100"
          leave-active-class="transition ease-in duration-75"
          leave-from-class="transform opacity-100 scale-100"
          leave-to-class="transform opacity-0 scale-95"
        >
          <div
            v-if="isMenuOpen"
            class="absolute right-0 mt-2 py-2 w-56 bg-white rounded-md shadow-xl z-30 text-gray-800 ring-1 ring-black ring-opacity-5"
            role="menu"
            aria-orientation="vertical"
            aria-labelledby="user-menu-button"
          >
            <template v-if="!isAdmin">
              <a
                href="#"
                @click.prevent="openLoginModal"
                class="flex items-center px-4 py-2 text-sm hover:bg-gray-100"
                role="menuitem"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2 text-gray-500" viewBox="0 0 20 20" fill="currentColor">
                  <path fill-rule="evenodd" d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z" clip-rule="evenodd" />
                </svg>
                Вход для администратора
              </a>
            </template>
            <template v-else>
              <!-- ОСТАВЛЯЕМ -->
              <a href="#" @click.prevent="triggerOpenAddPaintingModal" class="flex items-center px-4 py-2 text-sm hover:bg-gray-100" role="menuitem">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2 text-gray-500" viewBox="0 0 20 20" fill="currentColor">
                  <path d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm12 12H4l4-8 3 6 2-4 3 6z" />
                </svg>
                Добавить картину
              </a>
              <!-- УДАЛЕНО: Изменить картину -->
              <!-- УДАЛЕНО: Удалить картину -->
              
              <!-- ОСТАВЛЯЕМ КАК ЗАГЛУШКУ ДЛЯ БУДУЩЕГО -->
              <a href="#" @click.prevent="triggerOpenAnalyticsModal" class="flex items-center px-4 py-2 text-sm hover:bg-gray-100" role="menuitem">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2 text-gray-500" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M11 3.055A9.001 9.001 0 1020.945 13H11V3.055z" />
                  <path stroke-linecap="round" stroke-linejoin="round" d="M20.488 9H15V3.512A9.025 9.025 0 0120.488 9z" />
                </svg>
                Аналитика
              </a>
              <div class="border-t my-1 border-gray-200"></div>
              <a
                href="#"
                @click.prevent="triggerLogout"
                class="flex items-center px-4 py-2 text-sm hover:bg-gray-100 text-red-600 hover:text-red-700"
                role="menuitem"
              >
                <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 mr-2" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                  <path stroke-linecap="round" stroke-linejoin="round" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" />
                </svg>
                Выход
              </a>
            </template>
          </div>
        </transition>
      </div>
    </div>

    <LoginModal :show="showLoginModal" @close="closeLoginModal" @login-success="handleLoginSuccess" />
  </header>
</template>

<script>
import LoginModal from './LoginModal.vue';

export default {
  name: 'Header',
  components: {
    LoginModal,
  },
  props: {
    isAdmin: {
      type: Boolean,
      required: true,
    },
  },
  // Объявляем все события, которые компонент может генерировать
  emits: ['admin-login', 'admin-logout', 'open-add-painting-modal', 'open-analytics-modal'],
  data() {
    return {
      isMenuOpen: false,
      showLoginModal: false,
    };
  },
  methods: {
    toggleMenu() {
      this.isMenuOpen = !this.isMenuOpen;
    },
    closeMenu() {
      this.isMenuOpen = false;
    },
    openLoginModal() {
      this.showLoginModal = true;
      this.closeMenu();
    },
    closeLoginModal() {
      this.showLoginModal = false;
    },
    handleLoginSuccess() {
      this.$emit('admin-login');
      this.showLoginModal = false;
      this.closeMenu();
    },
    triggerLogout() {
      this.$emit('admin-logout');
      this.closeMenu();
    },
    triggerOpenAddPaintingModal() {
      this.$emit('open-add-painting-modal');
      this.closeMenu();
    },
    triggerOpenAnalyticsModal() { // Новый метод вместо handleAdminAction
      this.$emit('open-analytics-modal');
      this.closeMenu();
    },
    handleClickOutside(event) {
      if (this.isMenuOpen) {
        const menuButton = this.$el.querySelector('button[aria-haspopup="true"]');
        const dropdownMenu = this.$el.querySelector('div[role="menu"]');
        if (menuButton && !menuButton.contains(event.target) && dropdownMenu && !dropdownMenu.contains(event.target)) {
           this.closeMenu();
        }
      }
    }
  },
  mounted() {
    document.addEventListener('click', this.handleClickOutside, true);
  },
  beforeUnmount() {
    document.removeEventListener('click', this.handleClickOutside, true);
  },
};
</script>