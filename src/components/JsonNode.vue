<script setup>
import { ref, computed } from 'vue'

const props = defineProps({
  data: { required: true },
  nodeKey: { default: null },
  depth: { type: Number, default: 0 }
})

const open = ref(props.depth < 2)

const type = computed(() => {
  if (props.data === null) return 'null'
  if (Array.isArray(props.data)) return 'array'
  return typeof props.data
})

const isExpandable = computed(() => type.value === 'object' || type.value === 'array')

const entries = computed(() => {
  if (type.value === 'array') return props.data.map((v, i) => [i, v])
  if (type.value === 'object') return Object.entries(props.data)
  return []
})

const preview = computed(() => {
  if (type.value === 'array') return `[ ${props.data.length} items ]`
  if (type.value === 'object') return `{ ${Object.keys(props.data).length} keys }`
  return ''
})

function formatValue(val, t) {
  if (t === 'null') return 'null'
  if (t === 'string') return `"${val}"`
  return String(val)
}
</script>

<template>
  <span class="font-mono text-[13px] leading-[1.7]">

    <span v-if="nodeKey !== null" class="text-sky-300 mr-1">
      {{ typeof nodeKey === 'number' ? nodeKey : `"${nodeKey}"` }}:
    </span>

    <template v-if="isExpandable">
      <button
        @click="open = !open"
        class="text-[11px] text-slate-500 hover:text-slate-300 px-0.5 cursor-pointer bg-transparent border-none"
      >{{ open ? '▾' : '▸' }}</button>

      <span
        v-if="!open"
        @click="open = true"
        class="text-[12px] text-slate-600 hover:text-slate-400 ml-1 cursor-pointer"
      >{{ preview }}</span>

      <span v-else class="text-slate-500">{{ type === 'array' ? '[' : '{' }}</span>

      <div v-if="open" class="pl-5 border-l border-[#1e2130] ml-1.5">
        <div v-for="([k, v], i) in entries" :key="k" class="flex items-baseline">
          <JsonNode :data="v" :node-key="k" :depth="depth + 1" />
          <span v-if="i < entries.length - 1" class="text-slate-600">,</span>
        </div>
      </div>

      <span v-if="open" class="text-slate-500">{{ type === 'array' ? ']' : '}' }}</span>
    </template>

    <template v-else>
      <span :class="{
        'text-green-300':  type === 'string',
        'text-amber-400':  type === 'number',
        'text-purple-400': type === 'boolean',
        'text-red-400':    type === 'null',
      }">{{ formatValue(data, type) }}</span>
    </template>
  </span>
</template>
