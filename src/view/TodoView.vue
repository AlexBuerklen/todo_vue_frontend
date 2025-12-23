<script setup lang="ts">
import type { Todo } from "@/types/todo";
import { ref } from "vue";
import '@mdi/font/css/materialdesignicons.css'


const props = defineProps<{ todo: Todo[] }>();
const selectedTodo = ref<Todo | null>(null);
const drawer = ref(false);

function openTodo(todoItem: Todo) {
  selectedTodo.value = todoItem;
  drawer.value = true;
}

</script>

<template>
  <v-col v-if="props.todo.length === 0">
    Klick auf eine Kategorie, um die Todos anzuzeigen
  </v-col>

  <v-list v-else>
    <v-list-item v-for="todoItem in props.todo" :key="todoItem.id" @click="openTodo(todoItem)" style="cursor: pointer">
      <v-container fluid class="bg-grey-lighten-2 rounded-lg text-black">
        <v-row>Kategorie: {{ todoItem.category }}</v-row>
        <v-row>Titel: {{ todoItem.title }}</v-row>
        <v-row>Due to: {{ todoItem.due }}</v-row>
      </v-container>
    </v-list-item>
  </v-list>

  <v-navigation-drawer v-model="drawer" temporary location="right">
    
        <v-container v-if="selectedTodo" class="d-flex flex-column ga-2" style="cursor: pointer">

        <div class="d-flex flex-column ga-2">
            <v-hover v-slot="{ isHovering, props }">
                <v-sheet
                v-bind="props"
                class="pa-3 rounded-lg text-black"
                :class="isHovering ? 'bg-grey-lighten-2 elevation-2' : 'bg-grey-lighten-3'"
                >
                <div>Titel: {{ selectedTodo.title }}</div>
                </v-sheet>
            </v-hover>

            <v-hover v-slot="{ isHovering, props }">
                <v-sheet
                v-bind="props"
                class="pa-3 rounded-lg text-black"
                :class="isHovering ? 'bg-grey-lighten-2 elevation-2' : 'bg-grey-lighten-3'"
                >
                <div>Beschreibung: {{ selectedTodo.description }}</div>
                </v-sheet>
            </v-hover>

            <v-hover v-slot="{ isHovering, props }">
                <v-sheet
                v-bind="props"
                class="pa-3 rounded-lg text-black"
                :class="isHovering ? 'bg-grey-lighten-2 elevation-2' : 'bg-grey-lighten-3'"
                >
                <div>Fällig zum: {{ selectedTodo.due }}</div>
                </v-sheet>
            </v-hover>
        </div>

        </v-container>
    
  </v-navigation-drawer>
</template>