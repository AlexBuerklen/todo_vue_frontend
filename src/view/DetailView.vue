<script setup lang = "ts">
    import type { Todo } from "@/types/todo"

    defineProps<{ Todo: Todo[]}>()
    
</script>
<template>
    <v-col v-if="Todo.length === 0">
        Klick auf eine Kategorie, um die Todos anzuzeigen
    </v-col>
    <v-list v-else>
        <v-list-item v-for="todo in Todo" :key="todo.id">
            <v-container fluid class="bg-grey-lighten-2 rounded-lg text-black">
                <v-row> {{ todo.title }} </v-row>
                <v-row> Due to: {{ todo.due }}</v-row>
            </v-container>
        </v-list-item>
    </v-list>
</template>