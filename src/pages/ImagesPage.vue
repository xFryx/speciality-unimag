<template>
  <q-card class="image-generator-card">
    <q-tabs
      v-model="activeTab"
      dense
      class="text-primary"
      active-color="primary"
      indicator-color="primary"
      align="left"
    >
      <q-tab name="generator" label="Generador" icon="image" />
      <q-tab name="history" label="Historial" icon="photo_library" />
    </q-tabs>

    <q-separator />

    <q-tab-panels v-model="activeTab" animated>
      <!-- 🧠 Pestaña 1: Generador -->
      <q-tab-panel name="generator">
        <div class="row q-col-gutter-md">
          <!-- Formulario -->
          <div class="col-12 col-sm-6">
            <div class="text-h5 q-mb-md">Generador de Imágenes (OpenAI)</div>

            <q-input
              type="textarea"
              rows="4"
              filled
              v-model="prompt"
              label="Descripción de la imagen"
              @keyup.enter="generateImage"
              class="q-mb-md"
            />

            <q-select
              filled
              v-model="selectedModel"
              :options="modelOptions"
              label="Modelo"
              class="q-mb-md"
            />

            <q-select
              filled
              v-model="selectedSize"
              :options="sizeOptions"
              label="Tamaño"
              class="q-mb-md"
            />

            <q-select
              filled
              v-model="selectedQuality"
              :options="qualityOptions"
              label="Calidad"
              class="q-mb-md"
              v-if="selectedModel.value === 'dall-e-3'"
            />

            <q-btn
              label="Generar Imagen"
              color="primary"
              :loading="loading"
              @click="generateImage"
              class="q-mt-md full-width"
              push
            />
          </div>

          <!-- Imagen generada -->
          <div class="col-12 col-sm-6 flex flex-center">
            <div v-if="imageUrl" class="q-mt-md">
              <img
                style="height: 100%; width: 100%"
                :src="imageUrl"
                alt="Imagen generada"
                spinner-color="primary"
              />
            </div>
            <div v-else class="text-center q-mt-md">
              <q-banner dense class="bg-grey-3 text-dark q-pa-sm q-mb-sm rounded-borders">
                <q-icon name="warning" color="orange" class="q-mr-sm" />
                No hay imagen generada aún.
              </q-banner>
              <q-img
                src="/empty.png"
                alt="Sin imagen"
                style="max-width: 80%; max-height: 240px"
                spinner-color="grey"
              />
            </div>
          </div>
        </div>
      </q-tab-panel>

      <!-- 🖼️ Pestaña 2: Historial -->
      <q-tab-panel name="history">
        <div v-if="history.length > 0">
          <div class="text-h6 q-mb-md">Historial de Imágenes Generadas</div>
          <q-carousel
            v-model="slide"
            animated
            infinite
            control-color="primary"
            navigation
            thumbnails
            class="rounded-borders bg-grey-2"
          >
            <q-carousel-slide
              v-for="(img, index) in history"
              :key="index"
              :name="index"
              class="column items-center q-pa-md"
            >
              <q-card>
                <img
                  :src="img.url"
                  :alt="img.prompt"
                  style="max-height: 800px; width: 100%"
                  class="q-mb-sm"
                />
              </q-card>
              <div class="text-caption text-center q-mb-xs">
                <strong>Descripción:</strong> {{ img.prompt }}
              </div>
              <div class="text-caption text-center">
                <strong>Modelo:</strong> {{ img.model.label }} | <strong>Tamaño:</strong>
                {{ img.size.label }}
                <span v-if="img.model.value === 'dall-e-3'">
                  | <strong>Calidad:</strong> {{ img.quality.label }}
                </span>
              </div>
            </q-carousel-slide>
          </q-carousel>
        </div>
        <div v-else class="q-mt-md text-center">
          <q-icon name="photo_library" size="42px" color="grey-6" />
          <div class="q-mt-sm text-grey-7">Aún no has generado imágenes</div>
        </div>
      </q-tab-panel>
    </q-tab-panels>
  </q-card>
</template>

<script>
import { defineComponent, ref, getCurrentInstance } from 'vue'

export default defineComponent({
  name: 'ImageGenerator',
  setup() {
    const { proxy } = getCurrentInstance()

    const prompt = ref('')
    const selectedModel = ref({ label: 'DALL·E 3', value: 'dall-e-3' })
    const selectedSize = ref({ label: '1024x1024', value: '1024x1024' })
    const selectedQuality = ref({ label: 'HD', value: 'hd' })
    const imageUrl = ref('')
    const loading = ref(false)
    const history = ref([])
    const slide = ref(0)
    const activeTab = ref('generator')

    const modelOptions = [
      { label: 'DALL·E 2', value: 'dall-e-2' },
      { label: 'DALL·E 3', value: 'dall-e-3' },
    ]

    const sizeOptions = [
      { label: '256x256', value: '256x256' },
      { label: '512x512', value: '512x512' },
      { label: '1024x1024', value: '1024x1024' },
    ]

    const qualityOptions = [
      { label: 'Standard', value: 'standard' },
      { label: 'HD', value: 'hd' },
    ]

    const generateImage = async () => {
      if (!prompt.value) return

      loading.value = true
      imageUrl.value = ''

      try {
        const payload = {
          prompt: prompt.value,
          model: selectedModel.value.value,
          size: selectedSize.value.value,
        }

        if (selectedModel.value.value === 'dall-e-3') {
          payload.quality = selectedQuality.value.value
        }

        const response = await proxy.$api.post('testing/image-generator', payload)
        const url = response.data.data.url

        imageUrl.value = url

        history.value.unshift({
          url,
          prompt: prompt.value,
          model: { ...selectedModel.value },
          size: { ...selectedSize.value },
          quality: selectedModel.value.value === 'dall-e-3' ? { ...selectedQuality.value } : null,
        })

        slide.value = 0
        //activeTab.value = 'history'
      } catch (error) {
        console.error('Error generando la imagen:', error)
      }

      loading.value = false
    }

    return {
      prompt,
      selectedModel,
      selectedSize,
      selectedQuality,
      imageUrl,
      loading,
      modelOptions,
      sizeOptions,
      qualityOptions,
      generateImage,
      history,
      slide,
      activeTab,
    }
  },
})
</script>

<style scoped>
.image-generator-card {
  max-width: 1200px;
  margin: 20px auto;
  padding: 16px;
}
</style>
