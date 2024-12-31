<script setup lang="ts">
import { Icon } from '@iconify/vue';
import type { Ref } from 'vue';
import { onMounted, ref, watch } from 'vue';

const showModal: Ref<boolean> = ref(false);
const showAddNoteButton: Ref<boolean> = ref(true);
const newNote: Ref<string> = ref('');
const errorMessage: Ref<string> = ref('');

const notes: Ref<Note[]> = ref([]);
const singleNote: Ref<Note> = ref({
  id: -1,
  text: '',
  date: new Date(),
  backgroundColor: '',
});

interface Note {
  id: number;
  text: string;
  date: Date;
  backgroundColor: string;
}

watch(
  notes,
  (newVal: Note[]) => {
    localStorage.setItem('notes', JSON.stringify(newVal));
  },
  { deep: true }
);

const addNote = (): undefined | string => {
  if (hasErrorMessage()) {
    return;
  }

  notes.value.push({
    id: Math.floor(Math.random() * 1000000),
    text: newNote.value,
    date: new Date(),
    backgroundColor: getRandomColor(),
  });

  resetValues();
};

const editNote = (id: number): void => {
  showModal.value = true;
  showAddNoteButton.value = false;

  const existingNote = notes.value.find((note) => note.id === id);

  if (existingNote) {
    newNote.value = existingNote.text;
    singleNote.value = existingNote;
  }
};

const deleteNote = (id: number): void => {
  const isConfirmed: boolean = confirm(
    'Are you sure you want to delete this item?'
  );

  if (isConfirmed) {
    notes.value = notes.value.filter((item) => item.id !== id);
  }
};

const updateNote = (): undefined | string => {
  if (hasErrorMessage()) {
    return;
  }

  const existingNote = notes.value.find(
    (note) => note.id === singleNote.value.id
  );

  if (existingNote) {
    existingNote.text = newNote.value;
  }

  resetValues();
};

const getRandomColor = (): string => {
  var color = 'hsl(' + Math.random() * 360 + ', 100%, 75%)';
  return color;
};

const resetValues = () => {
  showAddNoteButton.value = true;
  showModal.value = false;
  newNote.value = '';
  errorMessage.value = '';
};

const hasErrorMessage = () => {
  if (newNote.value.length < 10) {
    return (errorMessage.value = 'Note needs to be 10 characters or more');
  }
};

onMounted(() => {
  const storedNotes = localStorage.getItem('notes');

  if (storedNotes) {
    notes.value = JSON.parse(storedNotes);
    for (let note of notes.value) {
      // Convert the date string back to a Date object
      note.date = new Date(note.date);
    }
  }
});
</script>

<template>
  <main>
    <div v-if="showModal" class="overlay">
      <div class="modal">
        <textarea
          v-model.trim="newNote"
          name="note"
          id="note"
          cols="30"
          rows="10"
        ></textarea>
        <p v-if="errorMessage">{{ errorMessage }}</p>
        <button v-if="showAddNoteButton" @click="addNote">Add Note</button>
        <button v-if="!showAddNoteButton" @click="updateNote">
          Update Note
        </button>

        <button class="close" @click="resetValues">Close</button>
      </div>
    </div>

    <div class="container">
      <header>
        <h1>Notes</h1>
        <button @click="showModal = true">+</button>
      </header>
      <div class="cards-container">
        <div
          v-for="note in notes"
          :key="note.id"
          class="card"
          :style="{ backgroundColor: note.backgroundColor }"
        >
          <p class="main-text">
            {{ note.text }}
          </p>
          <div class="card-bottom">
            <p class="date">{{ note.date.toLocaleDateString('en-US') }}</p>
            <div>
              <Icon
                icon="mdi:edit-outline"
                @click="editNote(note.id)"
                width="25"
              />
              <Icon
                icon="mdi:delete-outline"
                @click="deleteNote(note.id)"
                width="25"
              />
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
main {
  height: 100vh;
  width: 100vw;
}

.container {
  max-width: 1000px;
  padding: 10px;
  margin: 0 auto;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

h1 {
  font-weight: bold;
  margin-bottom: 25px;
  font-size: 75px;
}

header button {
  border: none;
  padding: 10px;
  width: 50px;
  height: 50px;
  cursor: pointer;
  background-color: rgb(21, 20, 20);
  border-radius: 100%;
  color: white;
  font-size: 20px;
}

.card {
  width: 225px;
  height: 225px;
  background-color: rgb(237, 182, 44);
  padding: 10px;
  border-radius: 15px;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  margin-right: 20px;
  margin-bottom: 20px;
}

.card-bottom {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.date {
  font-size: 13px;
  font-weight: bold;
}

.cards-container {
  display: flex;
  flex-wrap: wrap;
}

.overlay {
  position: absolute;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.77);
  z-index: 10;
  display: flex;
  align-items: center;
  justify-content: center;
}

.modal {
  width: 750px;
  background-color: white;
  border-radius: 10px;
  padding: 30px;
  position: relative;
  display: flex;
  flex-direction: column;
}

.modal button {
  padding: 10px 20px;
  font-size: 20px;
  width: 100%;
  background-color: blueviolet;
  border: none;
  color: white;
  cursor: pointer;
  margin-top: 15px;
}

.modal .close {
  background-color: rgb(193, 15, 15);
  margin-top: 7px;
}

.modal p {
  color: rgb(193, 15, 15);
}
</style>
