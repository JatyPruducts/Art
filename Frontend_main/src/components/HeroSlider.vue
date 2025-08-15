<template>
  <section class="relative h-[75vh] flex items-center justify-center bg-gray-900">
    <img :key="currentSlide" :src="slides[currentSlide]" alt="Слайд"
         class="absolute w-full h-full object-cover fade-in-out" />
    <div class="relative text-center text-white px-4">
      <h2 class="text-2xl md:text-3xl font-bold max-w-2xl mx-auto">
        Здесь вы найдёте уникальные произведения искусства, отражающие богатое наследие. Приглашаем вас окунуться в мир творчества и красоты.
      </h2>
    </div>
  </section>
</template>

<script>
export default {
  name: 'HeroSlider',
  data() {
    return {
      currentSlide: 0,
      slides: ['images/1.jpg', 'images/2.jpg', 'images/3.jpg']
    };
  },
  mounted() {
    this.interval = setInterval(() => {
      this.currentSlide = (this.currentSlide + 1) % this.slides.length;
    }, 7000);
  },
  beforeUnmount() {
    clearInterval(this.interval);
  }
};
</script>

<style scoped>
/* Стили, которые мы перенесли из App.vue */
@keyframes fadeInOut {
  0% { opacity: 0; }
  30% { opacity: 1; }
  60% { opacity: 1; }
  100% { opacity: 0; }
}
.fade-in-out {
  animation: fadeInOut 7s ease-in-out forwards;
  filter: brightness(60%);
}
</style>
