<script setup>
import { ref, computed } from 'vue'

// --- State for Variables and JCL Input ---
const tsoId = ref('')
const env = ref('')
const bcfDay = ref('')
const jclInput = ref('')
const editableInput = ref(null)

// --- Conversion Logic ---
const staticConversionRules = [
  { from: 'PDB2', to: 'TDB2' },
  { from: 'PAA3', to: 'DG11' }
  // Add more static rules here in the future
]

// Combine dynamic and static rules
const allConversionRules = computed(() => {
  const dynamicRules = [
    { from: 'TSO_ID', to: tsoId.value },
    { from: 'ENV', to: env.value },
    { from: 'BCF_DAY', to: bcfDay.value }
  ].filter(rule => rule.to) // Only include rules where a value is provided

  return [...dynamicRules, ...staticConversionRules]
})

function getHighlightedInputHtml() {
  if (!jclInput.value) return ''
  let output = jclInput.value
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')

  const fromTerms = allConversionRules.value.map(rule => rule.from).filter(Boolean).join('|')
  if (!fromTerms) return output

  const regex = new RegExp(`(${fromTerms})`, 'g')
  return output.replace(regex, '<span class="highlight-from">$1</span>')
}

const convertedJcl = computed(() => {
  if (!jclInput.value) return ''
  let output = jclInput.value
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')

  for (const rule of allConversionRules.value) {
    if (rule.from && rule.to) {
      const highlightSpan = `<span class="highlight-to">${rule.to}</span>`
      output = output.replace(new RegExp(rule.from, 'g'), highlightSpan)
    }
  }
  return output
})

// --- Event Handlers for Editable Div ---
function handleInput(e) {
  jclInput.value = e.target.innerText
}

function handleFocus(e) {
  if (jclInput.value) {
    e.target.innerText = jclInput.value
  }
}

function handleBlur(e) {
  if (jclInput.value) {
    e.target.innerHTML = getHighlightedInputHtml()
  }
}
</script>

<template>
  <div class="app-container">
    <header class="top-bar">
      <h1 class="title">JCL Convertor</h1>
      <div class="variable-inputs">
        <div class="input-group">
          <label for="tso_id">TSO_ID</label>
          <input id="tso_id" v-model="tsoId" type="text" />
        </div>
        <div class="input-group">
          <label for="env">ENV</label>
          <input id="env" v-model="env" type="text" />
        </div>
        <div class="input-group">
          <label for="bcf_day">BCF_DAY</label>
          <input id="bcf_day" v-model="bcfDay" type="text" />
        </div>
      </div>
    </header>

    <main class="converter-grid">
      <div
        ref="editableInput"
        contenteditable="true"
        class="io-panel input-panel"
        @input="handleInput"
        @focus="handleFocus"
        @blur="handleBlur"
        data-placeholder="Paste your production JCL here..."
      ></div>
      <div class="io-panel output-panel" v-html="convertedJcl" data-placeholder="Conversion result will appear here..."></div>
    </main>
  </div>
</template>

<style scoped>
html,
body,
#app {
  height: 100%;
  margin: 0;
  padding: 0;
}

.app-container {
  display: flex;
  flex-direction: column;
  height: 100vh;
  box-sizing: border-box;
}

.top-bar {
  display: flex;
  align-items: center;
  padding: 0.5rem 1rem;
  background-color: #f0f0f0;
  border-bottom: 1px solid #ccc;
}

.title {
  font-size: 1.25rem;
  font-weight: 600;
  margin-right: 2rem;
  color: #333;
}

.variable-inputs {
  display: flex;
  gap: 1.5rem;
}

.input-group {
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.input-group label {
  font-size: 0.9rem;
  font-weight: 500;
  color: #555;
}

.input-group input {
  border: 1px solid #ccc;
  border-radius: 4px;
  padding: 0.5rem;
  font-family: monospace;
  width: 120px; /* Give a default width */
}

.converter-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  flex-grow: 1;
  height: 100%;
  overflow-y: auto; /* Allows grid to scroll if needed, not the panels */
}

.io-panel {
  width: 100%;
  height: 100%;
  border: none;
  padding: 1rem;
  font-family: monospace;
  font-size: 1rem;
  box-sizing: border-box;
  background-color: #f7f7f7;
  color: #333;
  overflow-y: auto;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.input-panel {
  border-right: 1px solid #ccc;
}

.io-panel:focus, .input-group input:focus {
  outline: none;
  background-color: #fff;
  border-color: #007bff;
}

.input-panel[data-placeholder]:empty:before,
.output-panel[data-placeholder]:empty:before {
  content: attr(data-placeholder);
  color: #999;
  cursor: text;
}
</style>

<style>
/* Global style for highlight, not scoped */
.highlight-to {
  background-color: yellow;
  font-weight: bold;
}
.highlight-from {
  background-color: #d4eaff; /* Light blue */
  border-radius: 3px;
}
</style>
