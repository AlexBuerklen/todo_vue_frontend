<script setup lang="ts">
import type { Todo } from "@/types/todo";
import { computed, watchEffect, ref, watch } from "vue";
import axios from "axios";
import { Temporal } from "@js-temporal/polyfill"

const props = defineProps<{ todo: Todo[]; category: string | null; loading: boolean }>()
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
const emit = defineEmits<{
  (e: "drawer-change", open: boolean): void;
  (e: "todo-created"): void;
  (e: "todo-deleted"): void;
}>();
const addTodoDialog = ref(false);
const addTodoFormRef = ref<any>(null);
const addTodoFormValid = ref(false);
const newTodo = ref({
  title: "",
  due: Temporal.Now.plainDateISO().toString(), 
  description: "",
});
const todoRules = {
  required: (v: string) => !!v.trim() || "Pflichtfeld",
  min2: (v: string) => (v.trim().length ?? 0) >= 2 || "Mindestens 2 Zeichen",
  min5: (v: string) => (v?.trim().length ?? 0) >= 5 || "Mindestens 5 Zeichen",
  isoDate: (v: string) => {
    try {
      Temporal.PlainDate.from(v);
      return true;
    } catch {
      return "Bitte ein gültiges Datum wählen";
    }
  },
  notPast: (v: string) => {
    try {
      const date = Temporal.PlainDate.from(v);
      const today = Temporal.Now.plainDateISO();
      return Temporal.PlainDate.compare(date, today) >= 0 || "Datum darf nicht in der Vergangenheit liegen";
    } catch {
      return "Bitte ein gültiges Datum wählen";
    }
  },
};


watch(drawer, (val) => emit("drawer-change", val))

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
    await axios.post(baseUrl + `/api/todo/changeDueDate/${selectedTodo.value.id}/${currentDate.value}`);

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
    await axios.post( baseUrl + `/api/todo/changeTitle/${selectedTodo.value.id}/${newTitle}`);

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
    await axios.post(baseUrl + `/api/todo/changeDescription/${selectedTodo.value.id}/${newDescription}`);
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

//////////// Add Todo //////////////

function resetAddTodoForm() {
  newTodo.value = {
    title: "",
    due: Temporal.Now.plainDateISO().toString(),
    description: "",
  };

  addTodoFormRef.value.resetValidation();
  error.value = null;
}

function closeAddTodoDialog() {
  addTodoDialog.value = false;
  resetAddTodoForm();
}

function addTodo() {
  if (!props.category) {
    error.value = "Bitte zuerst eine Kategorie auswählen.";
    return;
  }

  error.value = null;
  resetAddTodoForm();
  addTodoDialog.value = true;
}

async function submitAddTodo() {
  if (!props.category) {
    error.value = "Kategorie fehlt. Bitte wähle eine Kategorie.";
    return;
  }

  const result = await addTodoFormRef.value.validate();
  if (!result.valid) return;

  const payload = {
    title: newTodo.value.title.trim(),
    description: newTodo.value.description.trim(),
    due: newTodo.value.due,          
    category: props.category,        
  };

  try {

    await axios.post(baseUrl + "/api/todo/saveTodo", payload);

    emit("todo-created");

    statusMessageText.value = "Todo erfolgreich erstellt";
    statusMessageColor.value = "success";
    statusMessage.value = true;


    closeAddTodoDialog();
  } catch {
    statusMessageText.value = "Todo konnte nicht erstellt werden. Bitte erneut versuchen.";
    statusMessageColor.value = "error";
    statusMessage.value = true;
  }
}

//////////// Add Todo //////////////

/////////// Delete Todo ////////////

async function deleteTodo(){
  if(!selectedTodo.value) return

  try {
  await axios.delete(baseUrl + `/api/todo/deleteTodo/${selectedTodo.value.id}`)

    statusMessageText.value = "Todo erfolgreich gelöscht";
    statusMessageColor.value = "success";
    statusMessage.value = true;

    drawer.value = false

    emit("todo-deleted");
    
  } catch {
    statusMessageText.value = "Todo konnte nicht gelöscht werden. Bitte erneut versuchen.";
    statusMessageColor.value = "error";
    statusMessage.value = true;
  }

}

/////////// Delete Todo ////////////

</script>

<template>
  <v-col v-if="!props.category">
    Klick auf eine Kategorie, um die Todos anzuzeigen
  </v-col>

  <v-col v-else-if="props.loading">
    Lade Todos für "{{ props.category }}"...
  </v-col>

  <v-container v-else-if="props.todo.length === 0">
    Keine Todos in der Kategorie "{{ props.category }}"
    <v-btn color="primary" @click="addTodo">Todo hinzufügen</v-btn>
  </v-container>

  <v-container v-else>Kategorie: {{ props.category }}
      <v-btn color="primary" @click="addTodo">Todo hinzufügen</v-btn>
  </v-container>

  <v-list>
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
        <v-col>
          <v-btn
            icon="mdi-close"
            class="bg-grey-lighten-3 text-black"
            @click="drawer = false"
          />
        </v-col>
        <v-col class="d-flex justify-center align-center">
          <v-btn color="error" @click="deleteTodo()">Todo löschen</v-btn>
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

  <v-dialog v-model="addTodoDialog" max-width="650">
    <v-card>
      <v-card-title class="d-flex align-center justify-space-between">
        <span>Neues Todo</span>
        <v-btn icon="mdi-close" variant="text" @click="closeAddTodoDialog" />
      </v-card-title>

      <v-card-subtitle v-if="props.category">
        Kategorie wird automatisch gesetzt: <strong>{{ props.category }}</strong>
      </v-card-subtitle>

      <v-card-text>
        <v-alert v-if="error" type="error" variant="tonal" class="mb-3">
          {{ error }}
        </v-alert>

        <v-form ref="addTodoFormRef" v-model="addTodoFormValid">
          <v-text-field
            v-model="newTodo.title"
            label="Titel"
            variant="outlined"
            :rules="[todoRules.required, todoRules.min2]"
            autocomplete="off"
            class="mb-3"
          />

          <v-text-field
            v-model="newTodo.due"
            label="Fälligkeitsdatum"
            variant="outlined"
            type="date"
            :rules="[todoRules.required, todoRules.isoDate, todoRules.notPast]"
            class="mb-3"
          />

          <v-textarea
            v-model="newTodo.description"
            label="Beschreibung"
            variant="outlined"
            auto-grow
            :rules="[todoRules.required, todoRules.min5]"
          />
        </v-form>
      </v-card-text>

      <v-card-actions class="justify-end">
        <v-btn variant="text" @click="closeAddTodoDialog">Abbrechen</v-btn>
        <v-btn color="primary" :disabled="!addTodoFormValid" @click="submitAddTodo">
          Speichern
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>