<script setup>
import { ref } from 'vue'
import JsonNode from './components/JsonNode.vue'

const rawInput = ref('')
const parsed = ref(null)
const error = ref('')

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
</script>

<template>
  <div class="layout">
    <header class="top-bar">
      <h1>JSON Viewer</h1>
    </header>

    <main class="content">
      <section class="input-side">
        <textarea
          v-model="rawInput"
          placeholder="Paste your JSON here..."
          @input="tryParse"
          spellcheck="false"
        />
        <p v-if="error" class="parse-error">{{ error }}</p>
      </section>

      <section class="viewer-side">
        <p v-if="!rawInput" class="placeholder-hint">
          Your formatted JSON will appear here.
        </p>
        <p v-else-if="error" class="placeholder-hint error">
          Fix the error to see the tree.
        </p>
        <div v-else class="tree">
          <JsonNode :data="parsed" />
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
.layout {
  display: flex;
  flex-direction: column;
  height: 100vh;
}

.top-bar {
  padding: 14px 24px;
  border-bottom: 1px solid #1e2130;
  flex-shrink: 0;
}

.top-bar h1 {
  font-size: 18px;
  font-weight: 600;
  color: #94a3b8;
  letter-spacing: 0.5px;
}

.content {
  display: flex;
  flex: 1;
  overflow: hidden;
}

.input-side,
.viewer-side {
  flex: 1;
  display: flex;
  flex-direction: column;
  overflow: auto;
  padding: 16px;
}

.input-side {
  border-right: 1px solid #1e2130;
}

textarea {
  flex: 1;
  background: #141720;
  color: #cbd5e1;
  border: 1px solid #1e2130;
  border-radius: 6px;
  padding: 14px;
  font-family: 'Cascadia Code', 'Fira Code', 'Consolas', monospace;
  font-size: 13px;
  line-height: 1.6;
  resize: none;
  outline: none;
  transition: border-color 0.2s;
}

textarea:focus {
  border-color: #3b4a6b;
}

.parse-error {
  margin-top: 8px;
  font-size: 12px;
  color: #f87171;
  font-family: monospace;
}

.placeholder-hint {
  color: #4a5568;
  font-size: 14px;
  margin: auto;
}

.placeholder-hint.error {
  color: #f87171;
}

.tree {
  padding: 4px 0;
}
</style>
