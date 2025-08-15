<template>
  <transition
    enter-active-class="ease-out duration-300"
    enter-from-class="opacity-0"
    enter-to-class="opacity-100"
    leave-active-class="ease-in duration-200"
    leave-from-class="opacity-100"
    leave-to-class="opacity-0"
  >
    <div v-if="show" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full flex justify-center items-center z-50 p-4" @click.self="$emit('close')">
      <transition
        enter-active-class="ease-out duration-300"
        enter-from-class="opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
        enter-to-class="opacity-100 translate-y-0 sm:scale-100"
        leave-active-class="ease-in duration-200"
        leave-from-class="opacity-100 translate-y-0 sm:scale-100"
        leave-to-class="opacity-0 translate-y-4 sm:translate-y-0 sm:scale-95"
      >
        <div class="bg-white p-6 sm:p-8 rounded-lg shadow-xl w-full max-w-3xl mx-4 my-8 max-h-[90vh] flex flex-col">
          <div class="flex-shrink-0 flex justify-between items-center mb-6">
            <h2 class="text-2xl font-bold text-gray-800">Аналитика по датам</h2>
            <button @click="$emit('close')" class="text-gray-400 hover:text-gray-600 transition">
              <span class="sr-only">Закрыть</span>
              <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
            </button>
          </div>
          
          <!-- Блок с выбором периода -->
          <div class="flex-shrink-0 mb-6">
            <div class="flex flex-wrap gap-2 border-b pb-4">
              <button
                v-for="period in periods"
                :key="period.days"
                @click="selectedPeriod = period.days"
                :class="[
                  'px-4 py-2 text-sm font-medium rounded-md transition',
                  selectedPeriod === period.days ? 'bg-blue-600 text-white' : 'bg-gray-200 text-gray-700 hover:bg-gray-300'
                ]"
              >{{ period.label }}</button>
            </div>
          </div>

          <!-- Блок со сводкой -->
          <div class="flex-shrink-0 grid grid-cols-1 md:grid-cols-2 gap-4 mb-6">
            <div class="bg-blue-50 p-4 rounded-lg">
              <p class="text-sm text-blue-800">Посещений за период</p>
              <p class="text-3xl font-bold text-blue-900">{{ summary.visits }}</p>
            </div>
            <div class="bg-green-50 p-4 rounded-lg">
              <p class="text-sm text-green-800">Заявок за период</p>
              <p class="text-3xl font-bold text-green-900">{{ summary.submissions }}</p>
            </div>
          </div>

          <!-- Таблица с детализацией -->
          <div class="flex-grow overflow-y-auto">
            <table class="min-w-full divide-y divide-gray-200">
              <thead class="bg-gray-50 sticky top-0">
                <tr>
                  <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Дата</th>
                  <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Посещения</th>
                  <th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Заявки</th>
                </tr>
              </thead>
              <tbody class="bg-white divide-y divide-gray-200">
                <tr v-if="detailedStats.length === 0">
                    <td colspan="3" class="px-6 py-4 text-center text-gray-500">Нет данных за выбранный период.</td>
                </tr>
                <tr v-for="stat in detailedStats" :key="stat.date">
                  <td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">{{ formatDate(stat.date) }}</td>
                  <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{{ stat.visits }}</td>
                  <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">{{ stat.submissions }}</td>
                </tr>
              </tbody>
            </table>
          </div>

        </div>
      </transition>
    </div>
  </transition>
</template>

<script>
export default {
  name: 'AnalyticsModal',
  props: {
    show: { type: Boolean, required: true },
    visitsData: { type: Object, default: () => ({}) },
    submissionsData: { type: Object, default: () => ({}) },
  },
  emits: ['close'],
  data() {
    return {
      selectedPeriod: 7, // По умолчанию показываем за 7 дней
      periods: [
        { label: 'Сегодня', days: 1 },
        { label: 'Вчера', days: 2 }, // Особый случай, обработаем в computed
        { label: '7 дней', days: 7 },
        { label: '30 дней', days: 30 },
        { label: 'Все время', days: Infinity },
      ],
    };
  },
  computed: {
    // Вычисляем статистику на основе выбранного периода
    filteredData() {
      const today = new Date();
      today.setHours(0, 0, 0, 0);

      let startDate = new Date(today);
      if (this.selectedPeriod === 2) { // Вчера
        startDate.setDate(today.getDate() - 1);
      } else if (this.selectedPeriod !== Infinity) {
        startDate.setDate(today.getDate() - (this.selectedPeriod - 1));
      } else {
        startDate = new Date('1970-01-01'); // Начало эпохи для "Все время"
      }
      
      const allDates = new Set([...Object.keys(this.visitsData), ...Object.keys(this.submissionsData)]);
      const stats = [];

      allDates.forEach(dateStr => {
        const entryDate = new Date(dateStr);
        // Для "Вчера" нам нужна точная дата, а не диапазон
        if (this.selectedPeriod === 2) {
            if(entryDate.getTime() === startDate.getTime()) {
                stats.push({
                    date: dateStr,
                    visits: this.visitsData[dateStr] || 0,
                    submissions: this.submissionsData[dateStr] || 0,
                });
            }
        } else if (entryDate >= startDate) {
           stats.push({
                date: dateStr,
                visits: this.visitsData[dateStr] || 0,
                submissions: this.submissionsData[dateStr] || 0,
            });
        }
      });
      
      // Сортируем по дате в порядке убывания (сначала новые)
      return stats.sort((a, b) => new Date(b.date) - new Date(a.date));
    },
    summary() {
      return this.filteredData.reduce((acc, curr) => {
        acc.visits += curr.visits;
        acc.submissions += curr.submissions;
        return acc;
      }, { visits: 0, submissions: 0 });
    },
    detailedStats() {
      return this.filteredData;
    }
  },
  methods: {
    formatDate(dateStr) {
      const date = new Date(dateStr);
      const today = new Date();
      today.setHours(0, 0, 0, 0);
      const yesterday = new Date(today);
      yesterday.setDate(today.getDate() - 1);

      if (date.getTime() === today.getTime()) return 'Сегодня';
      if (date.getTime() === yesterday.getTime()) return 'Вчера';
      
      return date.toLocaleDateString('ru-RU', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
    }
  }
};
</script>