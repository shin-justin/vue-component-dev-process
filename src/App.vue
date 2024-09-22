<script setup lang="ts">
import { ref } from 'vue';
import ContextMenu from './components/ContextMenu.vue';

const isMenuVisible = ref<boolean>(false);

const toggleMenu = () => (isMenuVisible.value = !isMenuVisible.value);

const itemValue = ref<string>('');
</script>

<template>
  <div>
    <ListRow />
    <button
      aria-haspopup="true"
      :aria-expanded="isMenuVisible"
      aria-controls="context-menu"
      aria-label="Toggle menu"
      @click="toggleMenu"
      class="button mb-2"
      type="button"
    >
      Show Menu
    </button>
    <ContextMenu
      :is-menu-visible="isMenuVisible"
      :items="['Latest', 'First']"
      @update:model-value="itemValue = $event"
    />
  </div>
</template>

<style scoped>
.mb-2 {
  margin-bottom: 7rem;
}
</style>
