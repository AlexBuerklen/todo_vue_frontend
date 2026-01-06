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
    const categories = ref<Category[]>([]);
    const selectedCategory = ref<string | null>(null)
    const todoCategoryFilteredList = ref<Todo[]>([]);

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
            const { data } = await axios.get<Todo[]>(baseUrl + `/api/Todo/getCategoryFilteredTodos/${category}`);
            todoCategoryFilteredList.value = data;
        } catch {
            error.value = `Todos mit der Kategorie ${category} konnten nicht geladen`
        } finally {
            loading.value = false;
        }
    }

    onMounted(() => {void loadTodoCategories()})
</script>

<template>
    <v-navigation-drawer permanent :width="300" color="blue-grey">
        <v-col>
            <v-card-title>Kategorien</v-card-title>
        </v-col>
        <v-divider></v-divider>

        <div v-if="loading">Lade...</div>
        <div v-else-if="error"> {{ error }}</div>

        <v-list v-else>
            <v-hover v-for="category in categoriesList" :key="category.id" v-slot="{ isHovering, props }">
                <v-list-item
                    v-bind="props"
                    :class="{ 'bg-grey-lighten-3': isHovering}"
                    style="cursor: pointer"
                >        
                    <v-col @click="loadContentFromCategory(category.category)">{{ category.category }}</v-col>
                </v-list-item>
            </v-hover>
        </v-list>
    </v-navigation-drawer>
    <DetailView 
    :todo="todoCategoryFilteredList"
    :category="selectedCategory"
    >
    </DetailView>
</template>
