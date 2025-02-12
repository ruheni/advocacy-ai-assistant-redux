<script setup>
import { onMounted, ref, computed } from 'vue'
import { useAppStore } from '@/stores/app'
import { storeToRefs } from 'pinia'
import promptService from '@/services/promptService'

const store = useAppStore()
const { userMessage, userId, prompts } = storeToRefs(store)

const categories = ref([])
const filterVal = ref('')
const filterBy = ref('')

onMounted(() => {
  getPrompts()
})

async function getPrompts() {
  try {
    // get all prompts from db
    store.prompts = await promptService.getPrompts()

    // get the distinct categories from the prompts result
    categories.value = [...new Set(prompts.value.map((prompt) => prompt.category))]
  } catch (error) {
    store.errorMessage = error.content
  }
}

function setUserMessage(message) {
  userMessage.value = message
}

const filteredPrompts = computed(() => {
  if (!filterVal.value) {
    return prompts.value
  }

  return store.prompts.filter((prompt) => prompt[filterBy.value] === filterVal.value)
})

async function deletePrompt(id) {
  await promptService.deletePrompt(id)
  store.prompts = store.prompts.filter((prompt) => prompt.id !== id)
}

function createPrompt() {
  store.initPromptState()
  store.showPromptEditor = true
}

</script>

<template>
  <article class="panel is-success">
    <div
      class="is-flex is-justify-content-space-between has-background-success has-text-white is-align-items-center panel-heading">
      <p class="">Prompt Library</p>
      <button class="button is-success" @click="createPrompt">
        <span class="icon">
          <i class="fas fa-plus"></i>
        </span>
      </button>
    </div>
    <p class="panel-tabs">
      <a :class="{ 'is-active': filterVal === '' }" @click="() => {
        filterVal = ''
        filterBy = 'category'
      }
        " v-if="categories.length > 0">All</a>
      <a v-for="category in categories" :key="category" @click="() => {
        filterVal = category
        filterBy = 'category'
      }
        " :class="{ 'is-active': filterVal === category }">
        {{ category }}</a>
      <a :class="{ 'is-active': filterVal === userId }" @click="() => {
        filterVal = userId
        filterBy = 'userId'
      }
        ">Mine</a>
    </p>
    <div v-for="prompt in filteredPrompts" :key="prompt.id"
      class="panel-block flex is-justify-content-space-between is-align-items-center">
      <div>
        <a @click="setUserMessage(prompt.text)">{{ prompt.title }}</a>
      </div>
      <div class="is-align-self-flex-end mr-3">
        <button class="delete" @click="deletePrompt(prompt.id)" v-if="prompt.userId === userId">Delete
        </button>
      </div>
    </div>
  </article>
</template>
