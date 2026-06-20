<script setup>
import { ref, computed, inject, watch } from 'vue'

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

const expandTick = inject('expandTick', ref(0))
const collapseTick = inject('collapseTick', ref(0))
const searchTerm = inject('searchTerm', ref(''))

watch(expandTick, () => { if (isExpandable.value) open.value = true })
watch(collapseTick, () => { if (isExpandable.value) open.value = false })

function subtreeHasMatch(data, term) {
  if (!term) return false
  const t = term.toLowerCase()

  if (data === null) return 'null'.includes(t)
  if (typeof data === 'string') return data.toLowerCase().includes(t)
  if (typeof data === 'number' || typeof data === 'boolean') return String(data).includes(t)

  if (Array.isArray(data)) {
    return data.some((v, i) => String(i).includes(t) || subtreeHasMatch(v, term))
  }
  if (typeof data === 'object') {
    return Object.entries(data).some(([k, v]) => k.toLowerCase().includes(t) || subtreeHasMatch(v, term))
  }
  return false
}

const isOpen = computed(() => {
  if (searchTerm.value && isExpandable.value) return true
  return open.value
})

function formatValue(val, t) {
  if (t === 'null') return 'null'
  if (t === 'string') return `"${val}"`
  return String(val)
}

const formattedKey = computed(() => {
  if (props.nodeKey === null) return ''
  return typeof props.nodeKey === 'number' ? String(props.nodeKey) : `"${props.nodeKey}"`
})

function highlight(text, term) {
  if (!term) return [{ text, match: false }]
  const lower = text.toLowerCase()
  const idx = lower.indexOf(term.toLowerCase())
  if (idx === -1) return [{ text, match: false }]
  return [
    { text: text.slice(0, idx), match: false },
    { text: text.slice(idx, idx + term.length), match: true },
    { text: text.slice(idx + term.length), match: false },
  ].filter(s => s.text !== '')
}
</script>

<template>
  <span class="font-mono text-[13px] leading-[1.7]">
    <span v-if="nodeKey !== null" class="text-sky-300 mr-1">
      <template v-for="seg in highlight(formattedKey, searchTerm)">
        <mark v-if="seg.match" class="bg-yellow-400/25 text-yellow-200 rounded-sm">{{ seg.text }}</mark>
        <span v-else>{{ seg.text }}</span>
      </template>:
    </span>

    <template v-if="isExpandable">
      <button
        @click="open = !open"
        class="text-[11px] text-slate-500 hover:text-slate-300 px-0.5 cursor-pointer bg-transparent border-none"
      >{{ isOpen ? '▾' : '▸' }}</button>

      <span
        v-if="!isOpen"
        @click="open = true"
        class="text-[12px] text-slate-600 hover:text-slate-400 ml-1 cursor-pointer"
      >{{ preview }}</span>

      <span v-else class="text-slate-500">{{ type === 'array' ? '[' : '{' }}</span>

      <div v-if="isOpen" class="pl-5 border-l border-[#1e2130] ml-1.5">
        <div v-for="([k, v], i) in entries" :key="k" class="flex items-baseline">
          <JsonNode :data="v" :node-key="k" :depth="depth + 1" />
          <span v-if="i < entries.length - 1" class="text-slate-600">,</span>
        </div>
      </div>

      <span v-if="isOpen" class="text-slate-500">{{ type === 'array' ? ']' : '}' }}</span>
    </template>

    <template v-else>
      <span :class="{
        'text-green-300':  type === 'string',
        'text-amber-400':  type === 'number',
        'text-purple-400': type === 'boolean',
        'text-red-400':    type === 'null',
      }">
        <template v-for="seg in highlight(formatValue(data, type), searchTerm)">
          <mark v-if="seg.match" class="bg-yellow-400/25 text-yellow-200 rounded-sm">{{ seg.text }}</mark>
          <span v-else>{{ seg.text }}</span>
        </template>
      </span>
    </template>
  </span>
</template>
