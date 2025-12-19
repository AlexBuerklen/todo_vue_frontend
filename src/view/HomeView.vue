<script setup lang="ts">
    import { ref, onMounted, onUnmounted } from "vue"
    import type { Todo } from "@/types/todo"
    import axios from "axios"
    import SideView from "./SideView.vue";

    // const todoList = ref<Todo[]>([])
    // const loading = ref(false)
    // const error = ref<string | null>(null)
    // const controller = new AbortController()
    // const baseUrl: string = "http://localhost:8080"

    // async function loadTodos(){
    //     loading.value = true
    //     error.value = null

    //     try {
    //         const { data } = await axios.get<Todo[]>(baseUrl + "/api/Todo/getAllTodos", {
    //             signal: controller.signal
    //         })
    //         todoList.value = data
    //         console.log("data: ", data)
    //     } catch (exception: any){
    //         if (exception?.name === "CanceledError") 
    //         return error.value = "Todos konnten nicht geladen werden."
    //     } finally {
    //         loading.value = false
    //     }
    // }

    // onMounted(() => { void loadTodos()})
    // onUnmounted(() => controller.abort())
</script>
<template>
    <SideView></SideView>

    <!-- <div v-if="loading">Lade...</div>
    <div v-else-if="error"> {{ error }}</div>

    <ul v-else>
        <li v-for="todo in todoList" :key="todo.id">
            <div>todo title: {{ todo.title }}</div>
            <div>todo description: {{ todo.description }}</div>
        </li>
    </ul> -->
</template>
