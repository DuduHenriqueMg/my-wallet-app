<script setup>
import { ref } from 'vue';

const props = defineProps({
  loading: Boolean,
});

const emit = defineEmits(['update', 'reset']);

function onChange() {
  emit('update');
}

function onReset() {
  emit('reset');
}
</script>

<template>
  <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg mb-6">
    <div class="p-6 border-b border-gray-200">
      <h2 class="text-xl font-semibold mb-4">Filtros</h2>
      <div class="grid grid-cols-1 md:grid-cols-4 gap-4">
        <slot :onChange="onChange" />
        <div class="flex items-end">
          <button 
            @click="onReset" 
            class="bg-gray-500 hover:bg-gray-700 text-white font-bold py-2 px-4 rounded transition-colors duration-200"
            :disabled="loading"
          >
            <span v-if="loading" class="inline-flex items-center">
              <svg class="animate-spin -ml-1 mr-3 h-5 w-5 text-white" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
              </svg>
              Carregando...
            </span>
            <span v-else>Limpar Filtros</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>