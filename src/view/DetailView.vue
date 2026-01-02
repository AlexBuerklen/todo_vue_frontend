<script setup lang="ts">
import type { Todo } from "@/types/todo";
import { computed, watchEffect, ref } from "vue";
import axios from "axios";
import { Temporal } from "@js-temporal/polyfill"

const props = defineProps<{ todo: Todo[], category: string | null}>();
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
const isEditingCalendar = ref(false);
const currentDate = ref<string>(Temporal.Now.plainDateISO().toString());
const monthName = computed(() => Temporal.PlainDate.from(currentDate.value).toLocaleString("de-DE", { month: "long", year: "numeric",}));
const statusMessage = ref(false);
const statusMessageText = ref("");
const statusMessageColor = ref("");
const editedDate = ref("");
const originalDate = ref("");

watchEffect(() => { selectedTodo.value ? selectedTodo.value.id : null; });

watchEffect(() => { editedTitle.value = selectedTodo.value ? selectedTodo.value.title : ""; });

watchEffect(() => {
  if (selectedTodo.value?.due) {
    editedDate.value = isoToGermanDate(selectedTodo.value.due);
    originalDate.value = editedDate.value;
    currentDate.value = selectedTodo.value.due;
  }
});


function openTodo(todoItem: Todo) {
  selectedTodo.value = todoItem;
  drawer.value = true;
}

///////////// Date /////////////////

function isoToGermanDate(date: string): string {
  return Temporal.PlainDate
    .from(date)
    .toLocaleString("de-DE", {
      day: "2-digit",
      month: "2-digit",
      year: "numeric",
    });
}

async function setDate() {
  if (!selectedTodo.value) return;

  try {
    await axios.post(baseUrl + `/api/Todo/changeDueDate/${selectedTodo.value.id}/${currentDate.value}`);

    selectedTodo.value.due = currentDate.value; 

    statusMessageText.value = "Datum erfolgreich geändert";
    statusMessage.value = true;
    statusMessageColor.value = "success";
  } catch {
    statusMessageText.value = "Datum konnte nicht geändert werden. Bitte erneut versuchen oder laden Sie die Seite neu.";
    statusMessage.value = true;
    statusMessageColor.value = "error";
  }
}

///////////// Date /////////////////

///////////// Title ////////////////

function cancelEditTitle(){
  if(!selectedTodo.value) return;

  editedTitle.value = originalTitle.value;
  isEditingTitle.value = false;
  error.value = null;
}

function startEditTitle() {
  if (!selectedTodo.value) return;

  originalTitle.value = selectedTodo.value.title;
  editedTitle.value = selectedTodo.value.title;
  isEditingTitle.value = true;
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
    await axios.post( baseUrl + `/api/Todo/changeTitle/${selectedTodo.value.id}/${newTitle}`);

    selectedTodo.value.title = newTitle;
    originalTitle.value = newTitle;

    isEditingTitle.value = false;

    statusMessageText.value = "Titel erfolgreich geändert";
    statusMessage.value = true;
    statusMessageColor.value = "success";

  } catch { 
    statusMessageText.value = "Titel konnte nicht geändert werden. Bitte erneut versuchen oder laden Sie die Seite neu."; 
    statusMessage.value = true;
    statusMessageColor.value = "error";
  }
}

///////////// Title ////////////////

//////////// Description ///////////

function cancelEditDescription(){
  if(!selectedTodo.value) return;

  editedDescription.value = originalDescription.value;
  isEditingDescription.value = false;
  error.value = null;
}

function startEditDescription() {
  if (!selectedTodo.value) return;

  originalDescription.value = selectedTodo.value.description;
  editedDescription.value = selectedTodo.value.description;
  isEditingDescription.value = true;
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

    statusMessageText.value = "Beschreibung erfolgreich geändert"
    statusMessage.value = true;
    statusMessageColor.value = "success";
  }
  catch { 
      statusMessageText.value = "Beschreibung konnte nicht geändert werden. Bitte erneut versuchen oder laden Sie die Seite neu.";
      statusMessage.value = true;
      statusMessageColor.value = "error";
  }
}

//////////// Description ///////////

//////////// Calendar //////////////

function openCalendar(){ isEditingCalendar.value = true; }

function shiftMonth(direction: 1 | -1) { currentDate.value = Temporal.PlainDate.from(currentDate.value).add({ months: direction }).toString(); }

function prevMonth() { shiftMonth(-1); }

function nextMonth() { shiftMonth(1); }

//////////// Calendar //////////////

</script>

<template>
  <v-col v-if="props.todo.length === 0">
    Klick auf eine Kategorie, um die Todos anzuzeigen
  </v-col>

  <v-list v-else>
    <v-container>Kategorie: {{ category }}</v-container>
    <v-list-item
      v-for="todoItem in props.todo"
      :key="todoItem.id"
      @click="openTodo(todoItem)"
      style="cursor: pointer"
    >
      <v-container fluid class="bg-grey-lighten-2 rounded-lg text-black">
        <v-row>Titel: {{ todoItem.title }}</v-row>
        <v-row>Fällig zum: {{ isoToGermanDate(todoItem.due) }}</v-row>
      </v-container>
    </v-list-item>
  </v-list>

  <v-navigation-drawer v-model="drawer" temporary location="right" :width="800">
    <v-container v-if="selectedTodo" class="d-flex flex-column ga-2">
      <v-row>
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
            hide-details
            autofocus
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
          <v-snackbar
          v-model="statusMessage"
          :timeout="3000"
          :color="statusMessageColor"
          location="bottom right"
          >
          {{ statusMessageText }}
          </v-snackbar>
        </v-sheet>
      </v-hover>

      <v-hover v-slot="{ isHovering, props: hoverProps }">
        <v-sheet
          v-bind="hoverProps"
          class="pa-3 rounded-lg text-black"
          :class="isHovering ? 'bg-grey-lighten-2 elevation-2' : 'bg-grey-lighten-3'"
          style="cursor: pointer;"
          @click="openCalendar()"
        >
          <div>Fällig zum: {{ editedDate }}</div>
        </v-sheet>
        <v-container v-if="isEditingCalendar">
            <v-toolbar >
              <v-btn
                icon="mdi-close"
                class="bg-grey-lighten-3 text-black"
                @click="isEditingCalendar = false"
              />
              <v-btn
                color="grey-darken-2"
                size: small
                variant="text"
                icon
                @click="prevMonth()"
                >
                <v-icon size="small">mdi-chevron-left</v-icon>
              </v-btn>

              <v-btn
                color="grey-darken-2"
                size: small
                variant="text"
                icon
                @click="nextMonth()"
                >
                <v-icon size="small">mdi-chevron-right</v-icon>
              </v-btn>

              <div> {{ monthName }}</div>
            </v-toolbar>
            
            <v-calendar v-model="currentDate" type="month" @click:date="setDate()"></v-calendar>
        </v-container>
      </v-hover>
    </v-container>
  </v-navigation-drawer>
</template>