<script setup>
import { ref, computed, watch } from 'vue'

const props = defineProps({
  data: { required: true }
})

const NODE_W = 200
const NODE_H = 48
const H_GAP = 80
const V_GAP = 14

const panX = ref(32)
const panY = ref(32)
const zoom = ref(1)
const grabbing = ref(false)
let dragStart = null

const openPaths = ref(new Set())

function toggleNode(path) {
  const s = new Set(openPaths.value)
  s.has(path) ? s.delete(path) : s.add(path)
  openPaths.value = s
}

function getType(v) {
  if (v === null) return 'null'
  if (Array.isArray(v)) return 'array'
  return typeof v
}

function valuePreview(v, type) {
  if (type === 'null') return 'null'
  if (type === 'string') return v.length > 22 ? `"${v.slice(0, 22)}…"` : `"${v}"`
  if (type === 'array') return `[ ${v.length} items ]`
  if (type === 'object') return `{ ${Object.keys(v).length} keys }`
  return String(v)
}

const TYPE_COLOR = {
  object: '#60a5fa',
  array: '#a78bfa',
  string: '#4ade80',
  number: '#fbbf24',
  boolean: '#c084fc',
  null: '#f87171',
}

function buildGraph(data) {
  const nodes = []
  const edges = []
  let row = 0

  function visit(val, key, path, depth) {
    const type = getType(val)
    const isExpandable = type === 'object' || type === 'array'
    const isOpen = depth === 0 || openPaths.value.has(path)
    const id = nodes.length

    const node = {
      id, path, depth, type, isExpandable, isOpen,
      key: key === null ? 'root' : String(key),
      preview: valuePreview(val, type),
      color: TYPE_COLOR[type] ?? '#64748b',
      x: depth * (NODE_W + H_GAP),
      y: 0,
    }
    nodes.push(node)

    const childIds = []

    if (isExpandable && isOpen) {
      const entries = type === 'array'
        ? val.slice(0, 50).map((v, i) => [String(i), v])
        : Object.entries(val).slice(0, 50)

      for (const [k, v] of entries) {
        childIds.push(visit(v, k, `${path}.${k}`, depth + 1))
      }
    }

    if (childIds.length === 0) {
      node.y = row * (NODE_H + V_GAP)
      row++
    } else {
      node.y = (nodes[childIds[0]].y + nodes[childIds[childIds.length - 1]].y) / 2
    }

    for (const cid of childIds) {
      edges.push({ from: node, to: nodes[cid] })
    }

    return id
  }

  visit(data, null, 'root', 0)
  return { nodes, edges }
}

const graph = computed(() => buildGraph(props.data))

function edgePath(e) {
  const x1 = e.from.x + NODE_W
  const y1 = e.from.y + NODE_H / 2
  const x2 = e.to.x
  const y2 = e.to.y + NODE_H / 2
  const cx = (x1 + x2) / 2
  return `M ${x1} ${y1} C ${cx} ${y1} ${cx} ${y2} ${x2} ${y2}`
}

function truncate(str, max) {
  return str.length > max ? str.slice(0, max) + '…' : str
}

function onMouseDown(e) {
  grabbing.value = true
  dragStart = { mx: e.clientX, my: e.clientY, px: panX.value, py: panY.value }
}
function onMouseMove(e) {
  if (!dragStart) return
  panX.value = dragStart.px + (e.clientX - dragStart.mx)
  panY.value = dragStart.py + (e.clientY - dragStart.my)
}
function onMouseUp() {
  grabbing.value = false
  dragStart = null
}
function onWheel(e) {
  e.preventDefault()
  const factor = e.deltaY > 0 ? 0.92 : 1.08
  zoom.value = Math.min(3, Math.max(0.15, zoom.value * factor))
}

watch(() => props.data, () => {
  panX.value = 32
  panY.value = 32
  zoom.value = 1
  openPaths.value = new Set()
})
</script>

<template>
  <svg class="w-full h-full select-none" :style="{ cursor: grabbing ? 'grabbing' : 'grab' }" @mousedown="onMouseDown"
    @mousemove="onMouseMove" @mouseup="onMouseUp" @mouseleave="onMouseUp" @wheel.prevent="onWheel">
    <g :transform="`translate(${panX}, ${panY}) scale(${zoom})`">
      <path v-for="(e, i) in graph.edges" :key="i" :d="edgePath(e)" fill="none" stroke="#2e3a50" stroke-width="1.5" />

      <g v-for="node in graph.nodes" :key="node.id" :transform="`translate(${node.x}, ${node.y})`"
        :style="{ cursor: node.isExpandable ? 'pointer' : 'default' }" @mousedown.stop
        @click="node.isExpandable && toggleNode(node.path)">
        <rect :width="NODE_W" :height="NODE_H" rx="6" fill="#141720" :stroke="node.color" stroke-width="1.5"
          stroke-opacity="0.45" />
        <text x="12" y="18" font-size="11" font-family="monospace" fill="#7dd3fc">
          {{ truncate(node.key, 24) }}
        </text>
        <text x="12" y="34" font-size="11" font-family="monospace" :fill="node.color">
          {{ truncate(node.preview, 24) }}
        </text>

        <text v-if="node.isExpandable" :x="NODE_W - 12" y="30" font-size="14" font-family="monospace" :fill="node.color"
          text-anchor="middle" style="pointer-events: none">{{ node.isOpen ? '−' : '+' }}</text>
      </g>
    </g>
  </svg>
</template>
