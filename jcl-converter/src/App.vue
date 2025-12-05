<script setup>
import { ref, computed } from 'vue'

const jclInput = ref('')

const conversionRules = [
  { from: 'PDB2', to: 'TDB2' },
  { from: 'PAA3', to: 'DG11' }
  // Add more rules here in the future
]

const convertedJcl = computed(() => {
  let output = jclInput.value
  for (const rule of conversionRules) {
    // Use a regular expression with the 'g' flag to replace all occurrences.
    output = output.replace(new RegExp(rule.from, 'g'), rule.to)
  }
  return output
})
</script>

<template>

  <div class="app-container">

    <main class="converter-grid">

      <textarea

        v-model="jclInput"

        placeholder="Paste your production JCL here..."

        class="jcl-textarea"

      ></textarea>

      <textarea

        :value="convertedJcl"

        placeholder="Conversion result will appear here..."

        readonly

        class="jcl-textarea"

      ></textarea>

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



.converter-grid {

  display: grid;

  grid-template-columns: 1fr 1fr;

  flex-grow: 1;

  height: 100%;

}



.jcl-textarea {

  width: 100%;

  height: 100%;

  border: none;

  padding: 1rem;

  font-family: monospace;

  font-size: 1rem;

  box-sizing: border-box;

  resize: none;

  background-color: #f7f7f7;

  color: #333;

}



.jcl-textarea:first-child {

  border-right: 1px solid #ccc;

}



.jcl-textarea:focus {

  outline: none;

  background-color: #fff;

}

</style>
