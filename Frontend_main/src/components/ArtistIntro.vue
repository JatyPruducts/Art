<template>
  <section class="bg-[#1f3438] text-white py-12">
    <div ref="introRef"
         :class="['container mx-auto px-4 flex flex-col md:flex-row items-center fade-in', { visible }]">
      <img src="/images/face.jpg" alt="Художник"
           class="w-full md:w-1/3 h-auto object-cover rounded-lg shadow-lg" />
      <div class="md:ml-8 mt-8 md:mt-0 text-center md:text-left">
        <h2 class="text-3xl font-bold mb-4">Представляем галерею выдающегося художника Сабанова Степана Леонтьевича</h2>
        <p class="text-lg">Заслуженного художника РФ, народного художника РСО-Алания.</p>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  name: 'ArtistIntro',
  data() {
    return { visible: false };
  },
  mounted() {
    const observer = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) {
        this.visible = true;
        observer.disconnect();
      }
    }, { threshold: 0.3 });
    observer.observe(this.$refs.introRef);
  }
};
</script>

<style scoped>
/* Стили, которые мы перенесли из App.vue */
.fade-in {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 1s ease-out, transform 1s ease-out;
}
.fade-in.visible {
  opacity: 1;
  transform: translateY(0);
}
</style>