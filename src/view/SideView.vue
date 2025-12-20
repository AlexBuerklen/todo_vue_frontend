<script setup lang="ts">
    import { onMounted, ref } from "vue"
    import axios from 'axios'

    const todoCategories = ref<string[]>([]);
    const loading = ref(false);
    const error = ref<string | null>(null);
    const baseUrl = "http://localhost:8080"


    async function loadTodoTitle() {
        loading.value = true;
        error.value = null;   

        try {
            const { data } = await axios.get<string[]>(baseUrl + "/api/Todo/getTodoTitle");
                todoCategories.value = data;
        } catch {
            error.value = "Todos mit Titel Liste konnte nicht geladen werden.";
        } finally {
            loading.value = false;
        }
}

    onMounted(() => { void loadTodoTitle()})
</script>

<template>
    <v-navigation-drawer permanent :width="300" color="blue-grey">
        <v-card-title>Todos Kategorien</v-card-title>
        <v-divider></v-divider>

        <div v-if="loading">Lade...</div>
        <div v-else-if="error"> {{ error }}</div>

        <v-list v-else>
            <v-list-item v-for="category in todoCategories" :key="category">
                <v-col>{{ category }}</v-col>
            </v-list-item>
        </v-list>

    </v-navigation-drawer>
</template>
