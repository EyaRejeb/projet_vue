<script setup>
import ParticipationForm from "@/components/ParticipationForm.vue";
</script>

<template>
  <div class="participation">
    <ParticipationForm />
  </div>
</template>

<style scoped>
@media (min-width: 1024px) {
  .participation {
    min-height: 100vh;
    display: flex;
    align-items: center;
  }
}
</style>
