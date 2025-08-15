<template>
  <div v-if="show" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full flex justify-center items-center z-50" @click.self="closeModal">
    <div class="bg-white p-8 rounded-lg shadow-xl w-full max-w-md mx-4">
      <h2 class="text-2xl font-bold mb-6 text-gray-800 text-center">Авторизация администратора</h2>
      <form @submit.prevent="handleSubmit">
        <div class="mb-4">
          <label for="username" class="block text-gray-700 text-sm font-bold mb-2">Логин:</label>
          <input
            type="text"
            id="username"
            v-model="username"
            class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 leading-tight focus:outline-none focus:shadow-outline"
            required
          />
        </div>
        <div class="mb-6">
          <label for="password" class="block text-gray-700 text-sm font-bold mb-2">Пароль:</label>
          <input
            type="password"
            id="password"
            v-model="password"
            class="shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 mb-3 leading-tight focus:outline-none focus:shadow-outline"
            required
          />
        </div>
        <p v-if="error" class="text-red-500 text-xs italic mb-4">{{ error }}</p>
        <div class="flex items-center justify-between">
          <button
            type="submit"
            class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded focus:outline-none focus:shadow-outline"
          >
            Войти
          </button>
          <button
            type="button"
            @click="closeModal"
            class="inline-block align-baseline font-bold text-sm text-blue-500 hover:text-blue-800"
          >
            Отмена
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'LoginModal',
  props: {
    show: {
      type: Boolean,
      required: true,
    },
  },
  data() {
    return {
      username: '',
      password: '',
      error: '',
    };
  },
  methods: {
    async handleSubmit() {
      try {
        const formData = new URLSearchParams();
        formData.append('username', this.username);
        formData.append('password', this.password);
        console.log(this.username, this.password)
        const response = await axios.post(`http://127.0.0.1:8000/api/auth/login`, formData, {
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded'
          }
        });

        if (response.data.access_token) {
          this.error = '';
          
          // Сохраняем токен
          localStorage.setItem('access_token', response.data.access_token);
          
          // Устанавливаем токен для будущих запросов
          axios.defaults.headers.common['Authorization'] = `Bearer ${response.data.access_token}`;
          
          this.$emit('login-success');
        } else {
          this.error = 'Не удалось получить токен доступа';
        }
      }
      catch (error) {
        if (error.response && error.response.status === 401) {
          this.error = 'Неверный логин или пароль';
        } else {
          this.error = 'Ошибка сервера или сети';
          console.error("Ошибка запроса:", error);
        }
      }
    },
    closeModal() {
      this.username = '';
      this.password = '';
      this.error = '';
      this.$emit('close'); // Сообщаем родительскому компоненту о закрытии
    },
  },
  watch: {
    // Сбрасываем форму при скрытии модального окна
    show(newVal) {
      if (!newVal) {
        this.username = '';
        this.password = '';
        this.error = '';
      }
    }
  }
};
</script>