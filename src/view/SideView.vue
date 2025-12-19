<script setup lang="ts">
    import { onMounted, onUnmounted, ref } from "vue"
    import type { TodoTitle } from "@/types/todoTitle"
    import axios from 'axios'

    const todoListTitle = ref<TodoTitle[]>([]);
    const loading = ref(false);
    const error = ref<string | null>(null);
    const controller = new AbortController();
    const baseUrl = "http://localhost:8080"


    async function loadTodoTitle(){
        loading.value = true;
        error.value = null;

        try {
            const { data } = await axios.get<TodoTitle[]>(baseUrl + "/api/Todo/getTodoTitle", {
                signal: controller.signal
            })
            todoListTitle.value = data
            console.log(data)
        } catch (exception: any){
            if (exception?.name === "CanceledError")
            return error.value = "Todos mit Titel Liste konnte nicht geladen werden."
        } finally {
            loading.value = false
        }
    }

    onMounted(() => { void loadTodoTitle()})
    onUnmounted(() => controller.abort())
</script>

<template>
    <v-navigation-drawer permanent :width="360">
        <v-list-item title="Todo"></v-list-item>
        <v-divider></v-divider>

        <div v-if="loading">Lade...</div>
        <div v-else-if="error"> {{ error }}</div>

        <ul v-else>
            <li v-for="todo in todoListTitle" :key="todo.id">
                <div>{{ todo.title }}</div>
            </li>
        </ul>

    </v-navigation-drawer>
</template>
