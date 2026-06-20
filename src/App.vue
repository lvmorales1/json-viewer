<script setup>
import { ref, provide } from 'vue'
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
provide('expandTick', expandTick)
provide('collapseTick', collapseTick)

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

      <section class="flex flex-col flex-1 p-4 overflow-auto">
        <div v-if="parsed" class="flex justify-end gap-2 mb-2.5 shrink-0">
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

        <p v-if="!rawInput" class="m-auto text-sm text-slate-600">
          Your formatted JSON will appear here.
        </p>
        <p v-else-if="error" class="m-auto text-sm text-red-400">
          Fix the error to see the tree.
        </p>
        <div v-else class="py-1">
          <JsonNode :data="parsed" />
        </div>
      </section>

    </main>
  </div>
</template>
