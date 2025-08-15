<template>
  <section ref="callbackForm" class="relative bg-gradient-to-r from-gray-900 via-black to-gray-900 text-white py-16">
    <div class="relative container mx-auto px-4 max-w-xl">
      <div v-if="!submitted">
        <h2 class="text-3xl md:text-4xl font-bold mb-6 text-center">Перезвоните мне</h2>
        <p class="text-center mb-8">
          Оставьте артикул картины, ваш телефон и имя — мы свяжемся в кратчайшие сроки!
        </p>
        <form @submit.prevent="submit" class="space-y-4">
          <!-- Артикул -->
          <div>
            <label for="art" class="block text-sm font-medium mb-1">Артикул картины</label>
            <input type="text" id="art" name="art"
                   v-model="form.art"
                   required
                   class="w-full px-4 py-2 bg-gray-800 rounded-lg focus:ring-2 focus:ring-blue-500 outline-none"
                   placeholder="Например: 0123" />
          </div>
          <!-- Телефон -->
          <div>
            <label for="phone" class="block text-sm font-medium mb-1">Ваш телефон</label>
            <input type="tel" id="phone" name="phone"
                   v-model="form.phone"
                   required
                   :class="['w-full px-4 py-2 rounded-lg focus:ring-2 outline-none', errors.phone ? 'ring-red-500 bg-red-50 text-red-800' : 'bg-gray-800 focus:ring-blue-500']"
                   placeholder="+7 (___) ___-__-__" />
            <p v-if="errors.phone" class="mt-1 text-sm text-red-400">{{ errors.phone }}</p>
          </div>
          <!-- Имя -->
          <div>
            <label for="name" class="block text-sm font-medium mb-1">Ваше имя</label>
            <input type="text" id="name" name="name"
                   v-model="form.name"
                   required
                   :class="['w-full px-4 py-2 rounded-lg focus:ring-2 outline-none', errors.name ? 'ring-red-500 bg-red-50 text-red-800' : 'bg-gray-800 focus:ring-blue-500']"
                   placeholder="Иван Иванов" />
            <p v-if="errors.name" class="mt-1 text-sm text-red-400">{{ errors.name }}</p>
          </div>
          <div class="text-center">
            <button type="submit" :disabled="isFormInvalid"
                    class="mt-4 inline-block px-8 py-3 bg-blue-600 hover:bg-blue-700 rounded-full font-semibold transition-all disabled:opacity-50">
              Отправить заявку
            </button>
          </div>
        </form>
      </div>
      <div v-else class="text-center space-y-4">
        <h3 class="text-2xl font-bold">Спасибо!</h3>
        <p>Мы скоро вам перезвоним по указанному номеру.</p>
        <button @click="reset"
                class="mt-4 inline-block px-6 py-2 bg-gray-700 hover:bg-gray-600 rounded-full transition-all">
          Отправить ещё раз
        </button>
      </div>
    </div>
  </section>
</template>

<script>
import axios from 'axios';

export default {
  name: 'CallbackForm',
  props: ['preArt'],
  emits: ['form-submitted'],
  data() {
    return {
      form: { art: this.preArt || '', name: '', phone: '' },
      submitted: false,
      // Теперь errors может быть пустым по умолчанию, computed свойство все решит
      errors: { name: null, phone: null } 
    };
  },
  computed: {
    // --- Ключевое изменение ---
    // Это свойство будет true, если есть ХОТЯ БЫ ОДНА ошибка.
    isFormInvalid() {
      const nameIsInvalid = !this.form.name || !/\S+\s+\S+/.test(this.form.name.trim());
      const phoneIsInvalid = !this.form.phone || !/^(\+7|8)\s?\(?\d{3}\)?[\s-]?\d{3}[\s-]?\d{2}[\s-]?\d{2}$/.test(this.form.phone.trim());
      return nameIsInvalid || phoneIsInvalid;
    }
  },
  watch: {
    // Watch'еры теперь нужны только для отображения сообщений об ошибках, а не для блокировки кнопки
    preArt(newVal) {
      this.form.art = newVal;
    },
    'form.name'(value) {
      if (!value || !/\S+\s+\S+/.test(value.trim())) {
        this.errors.name = 'Введите имя и фамилию через пробел';
      } else {
        this.errors.name = null; // null или пустая строка
      }
    },
    'form.phone'(value) {
      if (!value || !/^(\+7|8)\s?\(?\d{3}\)?[\s-]?\d{3}[\s-]?\d{2}[\s-]?\d{2}$/.test(value.trim())) {
        this.errors.phone = 'Неверный формат телефона, например +7 (999) 123-45-67';
      } else {
        this.errors.phone = null;
      }
    }
  },
  methods: {
    // Метод submit теперь просто доверяет computed свойству
    async submit() {
      // Если форма невалидна, просто ничего не делаем.
      // Пользователь и так видит сообщения об ошибках.
      if (this.isFormInvalid) {
        // Можно дополнительно "потрясти" форму или подсветить все ошибки,
        // но для простоты просто выходим.
        return;
      }
      try {
        await axios.post(`http://127.0.0.1:8000/api/feedback`, {
          user_name: this.form.name,
          phone_number: this.form.phone,
          painting_id: this.form.art
        }); // Передача данных формы #чек
        console.log('Запрос отправлен:', this.form);
        this.submitted = true;
        this.$emit('form-submitted');
      }
      catch(error) 
      {
        console.error('Ошибка при отправке данных:', error);
        alert('Ошибка на сервере при отправке данных');
      }
    },

    reset() {
      this.form = { art: this.preArt || '', name: '', phone: '' };
      this.errors = { name: null, phone: null };
      this.submitted = false;
    }
  }
};
</script>
