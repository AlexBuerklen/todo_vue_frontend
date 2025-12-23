<script setup lang="ts">
    import type { Todo } from '@/types/todo'

    defineProps<{ todo: Todo[]}>()
</script>

<template>
    <v-list>
        <v-list-item v-for="todoItem in todo" :key="todoItem.id">
            <v-row> Kategorie: {{ todoItem.category }}</v-row>
            <v-row> Titel: {{ todoItem.title }} </v-row>
            <v-row> Due to: {{ todoItem.due }}</v-row>
        </v-list-item>
    </v-list>
</template>