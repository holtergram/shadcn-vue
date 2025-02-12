<script setup lang="ts">
import { useConfigStore } from '@/stores/config'
import { defineAsyncComponent } from 'vue'
import { Index } from '../../../__registry__'
import Spinner from './Spinner.vue'

const props = defineProps<{
  name: string
  typeName?: 'example' | 'block'
}>()
const { style } = useConfigStore()
const styleIndex = Index[style.value]

const componentRegistry = styleIndex[props.name]

const Component = defineAsyncComponent({
  loadingComponent: Spinner,
  loader: componentRegistry.component,
  timeout: 5000,
})
</script>

<template>
  <Component :is="Component" />
</template>
