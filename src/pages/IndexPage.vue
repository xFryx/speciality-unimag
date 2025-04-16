<template>
  <q-card class="translator-card">
    <q-tabs
      v-model="activeTab"
      dense
      class="text-grey"
      active-color="primary"
      indicator-color="primary"
      align="justify"
    >
      <q-tab name="translate" label="Traducir" />
      <q-tab name="history" label="Historial" />
    </q-tabs>

    <q-separator />

    <q-tab-panels v-model="activeTab" animated>
      <q-tab-panel name="translate">
        <div class="language-selector">
          <q-select
            v-model="sourceLang"
            :options="languageOptions"
            dense
            outlined
            style="width: 120px"
          />
          <q-btn flat icon="swap_horiz" @click="swapLanguages" class="swap-btn" />
          <q-select
            v-model="targetLang"
            :options="languageOptions"
            dense
            outlined
            style="width: 120px"
          />
        </div>

        <div class="text-container">
          <q-input
            v-model="inputText"
            type="textarea"
            outlined
            placeholder="Ingresa texto para traducir"
            :input-style="{ minHeight: '100px' }"
            @input="debouncedTranslate"
          />
          <q-input
            v-model="translatedText"
            type="textarea"
            outlined
            readonly
            placeholder="Traducción aparecerá aquí"
            :input-style="{ minHeight: '100px' }"
          />
        </div>

        <q-btn
          :loading="loading"
          color="primary"
          label="Traducir"
          @click="translateText"
          class="q-mt-md"
        />
      </q-tab-panel>

      <q-tab-panel name="history">
        <q-list>
          <q-item v-for="(item, index) in history" :key="index">
            <q-item-section>
              <q-item-label>{{ item.input }}</q-item-label>
              <q-item-label caption>{{ item.output }}</q-item-label>
            </q-item-section>
          </q-item>
        </q-list>
      </q-tab-panel>
    </q-tab-panels>
  </q-card>
</template>

<script>
import { defineComponent, ref, getCurrentInstance } from 'vue'
import { debounce } from 'quasar'

export default defineComponent({
  name: 'TranslatorPage',
  setup() {
    const { proxy } = getCurrentInstance()
    const activeTab = ref('translate')
    const sourceLang = ref()
    const targetLang = ref({ label: 'Inglés', value: 'Inglés' })
    const inputText = ref('')
    const translatedText = ref('')
    const history = ref([])

    const loading = ref(false)

    const languageOptions = [
      { label: 'Español', value: 'Español' },
      { label: 'Inglés', value: 'Inglés' },
      { label: 'Francés', value: 'Francés' },
    ]

    const translateText = async () => {
      if (!inputText.value) {
        translatedText.value = ''
        return
      }

      loading.value = true

      try {
        const response = await proxy.$api.post('testing/Legacy', {
          text: inputText.value,
          sourceLang: sourceLang.value.label,
          targetLang: targetLang.value.label,
        })

        console.log(response)

        const data = response.data.data.choices[0].message.content

        if (data) {
          translatedText.value = data
          history.value.unshift({
            input: inputText.value,
            output: translatedText.value,
          })
        }
      } catch (error) {
        console.log(error)
        translatedText.value = error.message
      }

      loading.value = false
    }

    const debouncedTranslate = debounce(translateText, 500)

    const swapLanguages = () => {
      const temp = sourceLang.value
      sourceLang.value = targetLang.value
      targetLang.value = temp
      if (inputText.value) {
        const tempText = inputText.value
        inputText.value = translatedText.value
        translatedText.value = tempText
        translateText()
      }
    }

    return {
      activeTab,
      sourceLang,
      targetLang,
      inputText,
      translatedText,
      history,
      languageOptions,
      translateText,
      debouncedTranslate,
      swapLanguages,
      loading,
    }
  },
})
</script>

<style scoped>
.translator-card {
  max-width: 600px;
  margin: 20px auto;
  padding: 16px;
}

.language-selector {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 16px;
}

.text-container {
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.swap-btn {
  padding: 0;
  min-width: 32px;
}
</style>
