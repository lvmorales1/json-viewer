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
  <span class="node">
    <span v-if="nodeKey !== null" class="key">
      {{ typeof nodeKey === 'number' ? nodeKey : `"${nodeKey}"` }}:&nbsp;
    </span>

    <template v-if="isExpandable">
      <button class="toggle" @click="open = !open">{{ open ? '▾' : '▸' }}</button>
      <span v-if="!open" class="preview" @click="open = true">{{ preview }}</span>
      <span v-else class="bracket">{{ type === 'array' ? '[' : '{' }}</span>

      <div v-if="open" class="children">
        <div v-for="([k, v], i) in entries" :key="k" class="child-line">
          <JsonNode :data="v" :node-key="k" :depth="depth + 1" />
          <span v-if="i < entries.length - 1" class="comma">,</span>
        </div>
      </div>

      <span v-if="open" class="bracket">{{ type === 'array' ? ']' : '}' }}</span>
    </template>

    <template v-else>
      <span :class="['value', type]">{{ formatValue(data, type) }}</span>
    </template>
  </span>
</template>

<style scoped>
.node {
  display: inline;
  font-family: 'Cascadia Code', 'Fira Code', Consolas, monospace;
  font-size: 13px;
  line-height: 1.7;
}

.key {
  color: #7dd3fc;
}

.toggle {
  background: none;
  border: none;
  color: #64748b;
  cursor: pointer;
  font-size: 11px;
  padding: 0 2px;
}

.toggle:hover { color: #94a3b8; }

.preview {
  color: #475569;
  cursor: pointer;
  font-size: 12px;
  margin-left: 4px;
}

.preview:hover { color: #94a3b8; }

.bracket { color: #64748b; }

.children {
  padding-left: 20px;
  border-left: 1px solid #1e2130;
  margin-left: 6px;
}

.child-line {
  display: flex;
  align-items: baseline;
}

.comma { color: #475569; }

.value.string  { color: #86efac; }
.value.number  { color: #fbbf24; }
.value.boolean { color: #c084fc; }
.value.null    { color: #f87171; }
</style>
