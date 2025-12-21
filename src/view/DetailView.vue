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
            <v-col>
                {{ todo }}
            </v-col>
        </v-list-item>
    </v-list>
</template>