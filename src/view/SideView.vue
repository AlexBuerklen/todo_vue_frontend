<script setup lang="ts">
    import { onMounted, ref } from "vue"
    import axios from 'axios'
    import DetailView from "./DetailView.vue";
    import type { Category } from "@/types/category";
    import type { Todo } from "@/types/todo"

    const categoriesList = ref<Category[]>([]);
    const loading = ref(false);
    const error = ref<string | null>(null);
    const baseUrl = "http://localhost:8080"
    const selectedCategory = ref<string | null>(null)
    const todoCategoryFilteredList = ref<Todo[]>([]);
    const showAddCategory = ref(false)
    const newCategoryText = ref("")
    const snackbar = ref(false)
    const snackbarText = ref("")
    const snackbarColor = ref<"success" | "error">("success")
    const loadingTodos = ref(false)
    const detailDrawerOpen = ref(false)

    function onTodoCreated() {
      if (!selectedCategory.value) return;
      void loadContentFromCategory(selectedCategory.value);
    }

    function onTodoDeleted(){
      if(!selectedCategory.value) return;
      void loadContentFromCategory(selectedCategory.value)
    }

    function openAddCategory() {
        showAddCategory.value = true
        error.value = null
    }

    function cancelAddCategory() {
        showAddCategory.value = false
        newCategoryText.value = ""
        error.value = null
    }

    async function saveNewCategory() {
        const name = newCategoryText.value.trim()
        if (!name) return

        try {
            await axios.post(baseUrl + `/api/category/saveCategory/${name}`)

            snackbarColor.value = "success"
            snackbarText.value = "Kategorie gespeichert."
            snackbar.value = true

            cancelAddCategory();

            void loadTodoCategories();

        } catch {
            snackbarColor.value = "error"
            snackbarText.value = "Kategorie konnte nicht gespeichert werden. Es sind keine doppelten Kategoriennamen erlaubt."
            snackbar.value = true
        }
    }

    async function loadTodoCategories() {
        loading.value = true;
        error.value = null;   

        try {
            const { data } = await axios.get<Category[]>(baseUrl + "/api/category/getAllCategories");
                categoriesList.value = data;
        } catch {
            error.value = "Kategorien konnten nicht geladen werden.";
        } finally {
            loading.value = false;
        }
    }

    async function loadContentFromCategory(category: string){
        selectedCategory.value = category;
        try {
            const { data } = await axios.get<Todo[]>(baseUrl + `/api/todo/getCategoryFilteredTodos/${category}`);
            todoCategoryFilteredList.value = data;
        } catch {
            error.value = `Todos mit der Kategorie ${category} konnten nicht geladen`
            todoCategoryFilteredList.value = []
        } finally {
            loading.value = false;
        }
    }

    onMounted(() => {loadTodoCategories()})
</script>

<template>
  <v-navigation-drawer permanent :width="300" color="blue-grey" :class="{ 'drawer-disabled': detailDrawerOpen }">
    <v-row justify="space-between">
      <v-col cols="auto">
        <v-card-title>Kategorien</v-card-title>
      </v-col>

      <v-col cols="auto">
        <v-btn
          icon="mdi-plus"
          variant="text"
          color="white"
          @click="openAddCategory"
        />
      </v-col>
    </v-row>

    <div v-show="showAddCategory" class="px-2 pb-2">
      <v-row>
        <v-col>
          <v-text-field
            v-model="newCategoryText"
            label="Neue Kategorie"
            variant="outlined"
            density="compact"
            hide-details
            @keydown.enter="saveNewCategory"
            @keydown.esc="cancelAddCategory"
            @blur="cancelAddCategory"
          />
        </v-col>

        <v-col cols="auto" class="pl-2">
          <v-btn @mousedown.prevent icon="mdi-check" variant="text" color="white" @click="saveNewCategory" />
        </v-col>

        <v-col cols="auto">
          <v-btn @mousedown.prevent icon="mdi-close" variant="text" color="white" @click="cancelAddCategory" />
        </v-col>
      </v-row>
      <v-snackbar v-model="snackbar" :timeout="3000" :color="snackbarColor"> {{ snackbarText }} </v-snackbar>
    </div>

    <v-divider></v-divider>

    <div v-if="loading">Lade...</div>
    <div v-else-if="error">{{ error }}</div>

    <v-list v-else>
      <v-hover v-for="category in categoriesList" :key="category.id" v-slot="{ isHovering, props }">
        <v-list-item
          v-bind="props"
          :class="{ 'bg-grey-lighten-3': isHovering }"
          style="cursor: pointer"
        >
          <v-col @click="loadContentFromCategory(category.category)">
            {{ category.category }}
          </v-col>
        </v-list-item>
      </v-hover>
    </v-list>
  </v-navigation-drawer>

  <DetailView
  :todo="todoCategoryFilteredList"
  :category="selectedCategory"
  :loading="loadingTodos"
  @drawer-change="detailDrawerOpen = $event"
  @todo-created="onTodoCreated"
  @todo-deleted="onTodoDeleted"
/>
</template>

<style scoped>
    .drawer-disabled {
    pointer-events: none;
    user-select: none;
    opacity: 0.9; 
    }
</style>


