<script setup>
import { ref, provide, watch, nextTick } from 'vue'
import JsonNode from './components/JsonNode.vue'

const rawInput = ref('')
const parsed = ref(null)
const error = ref('')
const copied = ref(false)

function tryParse() {
  error.value = ''
  parsed.value = null
  if (!rawInput.value.trim()) return

  try {
    parsed.value = JSON.parse(rawInput.value)
  } catch (e) {
    error.value = e.message
  }
}

const expandTick = ref(0)
const collapseTick = ref(0)
const searchTerm = ref('')
const viewerEl = ref(null)
provide('expandTick', expandTick)
provide('collapseTick', collapseTick)
provide('searchTerm', searchTerm)

let marks = []
const matchIndex = ref(0)
const matchCount = ref(0)

function applyActiveClass() {
  marks.forEach((m, i) => m.classList.toggle('mark-active', i === matchIndex.value))
}

function scrollToCurrent() {
  marks[matchIndex.value]?.scrollIntoView({ behavior: 'smooth', block: 'center' })
}

watch(searchTerm, async () => {
  await nextTick()
  marks = Array.from(viewerEl.value?.querySelectorAll('mark') ?? [])
  matchCount.value = marks.length
  matchIndex.value = 0
  applyActiveClass()
  scrollToCurrent()
})

function nextMatch() {
  if (!marks.length) return
  matchIndex.value = (matchIndex.value + 1) % marks.length
  applyActiveClass()
  scrollToCurrent()
}

function prevMatch() {
  if (!marks.length) return
  matchIndex.value = (matchIndex.value - 1 + marks.length) % marks.length
  applyActiveClass()
  scrollToCurrent()
}

async function copyJson() {
  if (!parsed.value) return
  await navigator.clipboard.writeText(JSON.stringify(parsed.value, null, 2))
  copied.value = true
  setTimeout(() => { copied.value = false }, 1800)
}
</script>

<template>
  <div class="flex flex-col h-screen bg-[#0f1117] text-slate-300">

    <header class="px-6 py-3.5 border-b border-[#1e2130] shrink-0">
      <h1 class="text-lg font-semibold text-slate-400 tracking-wide">JSON Viewer</h1>
    </header>

    <main class="flex flex-1 overflow-hidden">

      <section class="flex flex-col flex-1 p-4 border-r border-[#1e2130] overflow-auto">
        <textarea
          v-model="rawInput"
          placeholder="Paste your JSON here..."
          @input="tryParse"
          spellcheck="false"
          class="flex-1 resize-none outline-none rounded-md p-3.5
                bg-[#141720] text-slate-300 border border-[#1e2130]
                font-mono text-[13px] leading-relaxed
                focus:border-[#3b4a6b] transition-colors"
        />
        <p v-if="error" class="mt-2 text-xs text-red-400 font-mono">{{ error }}</p>
      </section>

      <section ref="viewerEl" class="flex flex-col flex-1 overflow-hidden">

        <div class="flex flex-col gap-2 px-4 pt-4 pb-2 border-b border-[#1e2130] shrink-0">
          <div v-if="parsed" class="flex items-center gap-2">
            <input
              v-model="searchTerm"
              placeholder="Search keys and values..."
              @keydown.enter="nextMatch"
              class="flex-1 bg-[#141720] text-slate-300 border border-[#1e2130]
                    rounded px-3 py-1 text-xs outline-none font-mono
                    focus:border-[#3b4a6b] transition-colors placeholder:text-slate-600"
            />
            <template v-if="searchTerm && matchCount > 0">
              <span class="text-xs text-slate-500 shrink-0">{{ matchIndex + 1 }} / {{ matchCount }}</span>
              <button @click="prevMatch" class="px-2 py-1 text-xs rounded border border-[#2e3a50] bg-[#1e2130] text-slate-400 hover:text-slate-200 hover:border-[#3b4a6b] transition-colors">↑</button>
              <button @click="nextMatch" class="px-2 py-1 text-xs rounded border border-[#2e3a50] bg-[#1e2130] text-slate-400 hover:text-slate-200 hover:border-[#3b4a6b] transition-colors">↓</button>
            </template>
            <span v-else-if="searchTerm && matchCount === 0" class="text-xs text-slate-600 shrink-0">no results</span>
          </div>

          <div v-if="parsed" class="flex justify-end gap-2">
            <button
              @click="expandTick++"
              class="px-3 py-1 text-xs rounded border transition-colors
                    text-slate-400 border-[#2e3a50] bg-[#1e2130]
                    hover:text-slate-200 hover:border-[#3b4a6b]"
            >Expand all</button>
            <button
              @click="collapseTick++"
              class="px-3 py-1 text-xs rounded border transition-colors
                    text-slate-400 border-[#2e3a50] bg-[#1e2130]
                    hover:text-slate-200 hover:border-[#3b4a6b]"
            >Collapse all</button>
            <button
              @click="copyJson"
              :class="[
                'px-3 py-1 text-xs rounded border transition-colors',
                copied
                  ? 'text-green-300 border-green-300/25'
                  : 'text-slate-400 border-[#2e3a50] bg-[#1e2130] hover:text-slate-200 hover:border-[#3b4a6b]'
              ]"
            >{{ copied ? 'Copied' : 'Copy' }}</button>
          </div>
        </div>

        <div class="flex-1 overflow-auto p-4">
          <p v-if="!rawInput" class="text-sm text-slate-600">
            Your formatted JSON will appear here.
          </p>
          <p v-else-if="error" class="text-sm text-red-400">
            Fix the error to see the tree.
          </p>
          <div v-else class="py-1">
            <JsonNode :data="parsed" />
          </div>
        </div>
      </section>

    </main>
  </div>
</template>
