<script setup>

import { onMounted, ref } from 'vue';

const props = defineProps({
  modelValue: [String, Number, null],
  disabled: Boolean
})

defineEmits(['update:modelValue']);

const input = ref(null);

onMounted(() => {
    if (input.value.hasAttribute('autofocus')) {
        input.value.focus();
    }
});

defineExpose({ focus: () => input.value.focus() });

</script>

<template>
    <select 
        ref="input"
        class="border-gray-300 focus:border-indigo-500 focus:ring-indigo-500 rounded-md shadow-sm mt-1 block w-full"
        :value="modelValue"
        @input="$emit('update:modelValue', $event.target.value)">
        <slot/>
    </select>
</template>
