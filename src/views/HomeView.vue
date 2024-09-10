<script setup>
import TheChat from '../components/TheChat.vue'
import TheHeader from '../components/TheHeader.vue'
</script>

<template>
  <header>
    <TheHeader></TheHeader>
  </header>
  <main>
    
    <section class="chat">
      <TheChat />
    </section>
  </main>
</template>
