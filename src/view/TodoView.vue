<script setup lang="ts">
import type { Todo } from "@/types/todo";
import { ref, watch } from "vue";
import axios from "axios";
import { ca } from "vuetify/locale";

const props = defineProps<{ todo: Todo[] }>();
const selectedTodo = ref<Todo | null>(null);
const drawer = ref(false);
const baseUrl = "http://localhost:8080";
const error = ref<string | null>(null);
const isEditingTitle = ref(false);
const editedTitle = ref("");
const originalTitle = ref("");
const isEditingDescription = ref(false);
const editedDescription = ref("");
const originalDescription = ref("");

watch(
  () => (selectedTodo.value ? selectedTodo.value.id : null),
  () => {editedTitle.value = selectedTodo.value ? selectedTodo.value.title : ""}
);

function openTodo(todoItem: Todo) {
  selectedTodo.value = todoItem;
  drawer.value = true;
}

function cancelEditTitle(){
  if(!selectedTodo.value) return;

  editedTitle.value = originalTitle.value;
  isEditingTitle.value = false;
  error.value = null;
}

function cancelEditDescription(){
  if(!selectedTodo.value) return;

  editedDescription.value = originalDescription.value;
  isEditingDescription.value = false;
  error.value = null;
}

function startEditTitle() {
  if (!selectedTodo.value) return;

  originalTitle.value = selectedTodo.value.title;
  editedTitle.value = selectedTodo.value.title;
  isEditingTitle.value = true;
}

function startEditDescription() {
  if (!selectedTodo.value) return;

  originalDescription.value = selectedTodo.value.description;
  editedDescription.value = selectedTodo.value.description;
  isEditingDescription.value = true;
}

async function saveTitle() {
  if (!selectedTodo.value) return;

  const newTitle = editedTitle.value.trim();

  if (!newTitle) {
    error.value = "Titel darf nicht leer sein."
    cancelEditTitle;
    return;
  }

  error.value = null;

  try {
    await axios.post( baseUrl + `/api/Todo/changeTitle/${selectedTodo.value.id}/${(newTitle)}`);

    selectedTodo.value.title = newTitle;
    originalTitle.value = newTitle;

    isEditingTitle.value = false;
  } catch { error.value = "Titel konnte nicht geändert werden. Bitte erneut versuchen oder laden Sie die Seite neu."; }
}

async function saveDescription(){
  if(!selectedTodo.value) return;

  const newDescription = editedDescription.value.trim();

  if(!newDescription){
    error.value = "Beschreibung darf nicht leer sein."
    cancelEditDescription;
    return;
  }

  error.value = null;

  try {
    await axios.post(baseUrl + `/api/Todo/changeDescription/${selectedTodo.value.id}/${newDescription}`);
    selectedTodo.value.description = newDescription;
    originalDescription.value = newDescription;
    isEditingDescription.value = false;
  } catch { error.value = "Beschreibung konnte nicht geändert werden. Bitte erneut versuchen oder laden Sie die Seite neu."; }
}
</script>

<template>
  <v-col v-if="props.todo.length === 0">
    Klick auf eine Kategorie, um die Todos anzuzeigen
  </v-col>

  <v-list v-else>
    <v-list-item
      v-for="todoItem in props.todo"
      :key="todoItem.id"
      @click="openTodo(todoItem)"
      style="cursor: pointer"
    >
      <v-container fluid class="bg-grey-lighten-2 rounded-lg text-black">
        <v-row>Kategorie: {{ todoItem.category }}</v-row>
        <v-row>Titel: {{ todoItem.title }}</v-row>
        <v-row>Due to: {{ todoItem.due }}</v-row>
      </v-container>
    </v-list-item>
  </v-list>

  <v-navigation-drawer v-model="drawer" temporary location="right" :width="800">
    <v-container v-if="selectedTodo" class="d-flex flex-column ga-2">
      <v-row>
        <v-col class="text-h6">Kategorie: {{ selectedTodo.category }}</v-col>
        <v-col cols="auto">
          <v-btn
            icon="mdi-close"
            class="bg-grey-lighten-3 text-black"
            @click="drawer = false"
          />
        </v-col>
      </v-row>

      <v-alert v-if="error" type="error" variant="tonal">
        {{ error }}
      </v-alert>

      <v-hover v-slot="{ isHovering, props: hoverProps }">
        <v-sheet
          v-bind="hoverProps"
          class="pa-3 rounded-lg text-black"
          style="cursor: pointer"
          :class="isHovering ? 'bg-grey-lighten-2 elevation-2' : 'bg-grey-lighten-3'"
          @click="startEditTitle"
        >
          <div v-if="!isEditingTitle"> Titel: {{ selectedTodo.title }} </div>

          <v-text-field
            v-if="isEditingTitle"
            v-model="editedTitle"
            label="Titel bearbeiten"
            variant="outlined"
            density="compact"
            autofocus
            hide-details
            @click.stop
            @keydown.enter.prevent="saveTitle"
            @keydown.esc.prevent="cancelEditTitle"
            @blur="cancelEditTitle"
          />
        </v-sheet>
      </v-hover>

      <v-hover v-slot="{ isHovering, props: hoverProps }">
        <v-sheet
          v-bind="hoverProps"
          class="pa-3 rounded-lg text-black"
          :class="isHovering ? 'bg-grey-lighten-2 elevation-2' : 'bg-grey-lighten-3'"
          style="cursor: pointer"
          @click="startEditDescription"
        >

          <div v-if="!isEditingDescription">Beschreibung: {{ selectedTodo.description }}</div>

          <v-text-field
            v-if="isEditingDescription"
            v-model="editedDescription"
            label="Beschreibung bearbeiten"
            variant="outlined"
            density="compact"
            autofocus
            hide-details
            @click.stop
            @keydown.enter.prevent="saveDescription"
            @keydown.esc.prevent="cancelEditDescription"
            @blur="cancelEditDescription"
          />
        </v-sheet>
      </v-hover>

      <v-hover v-slot="{ isHovering, props: hoverProps }">
        <v-sheet
          v-bind="hoverProps"
          class="pa-3 rounded-lg text-black"
          :class="isHovering ? 'bg-grey-lighten-2 elevation-2' : 'bg-grey-lighten-3'"
        >
          <div>Fällig zum: {{ selectedTodo.due }}</div>
        </v-sheet>
      </v-hover>
    </v-container>
  </v-navigation-drawer>
</template>