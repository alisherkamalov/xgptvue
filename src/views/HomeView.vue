<script setup>
import TheWarning from '@/components/TheWarning.vue';
import TheChat from '../components/TheChat.vue'
import TheHeader from '../components/TheHeader.vue'
</script>

<template>
  <TheWarning/>
  <header>
    <TheHeader></TheHeader>
  </header>
  <main>
    <section class="chat">
      <TheChat />
    </section>
  </main>
</template>
