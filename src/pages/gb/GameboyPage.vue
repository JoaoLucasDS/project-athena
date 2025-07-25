<template>
  <q-page padding class="column items-center">
    <h5>Emulador Nostalgist.js no Quasar com uploader nativo</h5>

    <div
      ref="emulatorContainerRef"
      class="emulator-container"
      :class="{ 'emulator-hidden': !isRunning }"
    />

    <q-uploader
      v-if="!isRunning"
      url=""
      :auto-upload="false"
      label="Selecione a ROM (.gba)"
      accept=".gba"
      max-files="1"
      style="max-width: 400px"
      @added="onFilesAdded"
      no-thumbnails
      hide-upload-btn
      ref="uploader"
    />

    <q-row>
      <q-select
        v-if="!isRunning"
        v-model="selectedCategory"
        :options="categories"
        label="Categoria de shader"
        emit-value
        map-options
        class="q-mb-sm"
        style="width: 250px"
      />

      <q-select
        v-if="!isRunning && shaders.length > 0"
        v-model="selectedShader"
        :options="shaders"
        label="Shader"
        emit-value
        map-options
        style="width: 250px"
      />
    </q-row>

    <q-row v-if="isRunning">
      <q-btn flat round color="negative" icon="close" @click="clearEmulator" />
      <q-btn flat round color="primary" icon="fast_forward" @click="setFastFoward" />
    </q-row>
  </q-page>
</template>

<script setup>
import { ref, onBeforeUnmount, onMounted, watch } from 'vue'
import { Nostalgist } from 'nostalgist'
import shaderList from 'src/assets/shaders.json'

const selectedCategory = ref(null)
const selectedShader = ref(null)
const categories = ref([])
const shaders = ref([])

const emulatorContainerRef = ref(null)
let emulatorInstance = null
const uploader = ref(null)
const isRunning = ref(false)

onMounted(() => {
  categories.value = shaderList.categories.map((cat) => ({
    label: cat.name,
    value: cat.name,
    shaders: cat.shaders,
  }))
})

// Quando a categoria mudar, atualiza os shaders disponíveis
watch(selectedCategory, (newCategory) => {
  if (newCategory) {
    const cat = categories.value.find((c) => c.value === newCategory)
    shaders.value =
      cat?.shaders.map((shader) => ({
        label: shader.name,
        value: shader.file,
      })) || []
    selectedShader.value = null
  } else {
    shaders.value = []
    selectedShader.value = null
  }
})

function clearEmulator() {
  if (emulatorContainerRef.value) {
    emulatorContainerRef.value.innerHTML = ''
  }
  emulatorInstance = null
  isRunning.value = false
}

function setFastFoward() {
  emulatorInstance?.sendCommand('FAST_FORWARD')
}

async function onFilesAdded(files) {
  clearEmulator()

  if (!files || files.length === 0) return

  const file = files[0]
  const isValid = file.type === 'application/octet-stream' || file.name.endsWith('.gba')

  if (!isValid) {
    uploader.value.reset()
    return
  }

  try {
    const shaderFile = selectedShader.value || '' // fallback: nenhum shader
    const instance = await Nostalgist.launch({
      core: 'mgba',
      rom: file,
      shader: shaderFile,
    })

    emulatorInstance = instance
    const canvas = instance.getCanvas()

    if (emulatorContainerRef.value) {
      canvas.classList.add('emulator-canvas')
      emulatorContainerRef.value.appendChild(canvas)
    }

    isRunning.value = true
  } catch (error) {
    console.error('Erro ao iniciar o emulador com a ROM:', error)
    clearEmulator()
    uploader.value.reset()
  }
}

onBeforeUnmount(() => {
  clearEmulator()
})
</script>

<style>
/* Container responsivo com proporção GBA (240x160) */
.emulator-container {
  width: 100%;
  max-width: 475px; /* Largura máxima */
  aspect-ratio: 3/2; /* Proporção 3:2 (240x160) */
  border: 1px solid #ccc;
  margin-bottom: 1rem;
  display: flex;
  justify-content: center;
  background: #f0f0f0;
}

.emulator-canvas {
  position: relative !important;
  height: 100% !important;
}

.emulator-hidden {
  opacity: 0;
  pointer-events: none; /* desabilita interações */
  height: 0;
  overflow: hidden;
  /* ou display: none; mas aí perde animação */
}
</style>
