<template>
  <q-page class="flex flex-center">
    <!-- Contador -->
    <div class="row full-width items-center">
      <div class="col text-center">
        <q-btn
          @click="decreaseCounter"
          icon="remove"
          size="xl"
          round
          color="red"
        />
      </div>
      <div class="col text-center text-h2">
        {{ state.counter }}
      </div>
      <div class="col text-center">
        <q-btn
          @click="increaseCounter"
          icon="add"
          size="xl"
          round
          color="green"
        />
      </div>
    </div>

    <!-- Mensaje motivador -->
    <div class="row full-width items-center q-my-md">
      <div class="col text-center text-body1 text-black">
        {{ state.motivationalMessage }}
      </div>
    </div>

    <!-- Límites personalizables -->
    <div class="row full-width items-center q-my-md">
      <q-input
        v-model="state.minLimit"
        type="number"
        label="Límite mínimo"
        outlined
        class="col"
      />
      <q-input
        v-model="state.maxLimit"
        type="number"
        label="Límite máximo"
        outlined
        class="col"
      />
    </div>

    <!-- Historial de acciones -->
    <div class="row full-width q-my-md">
      <q-card class="full-width">
        <div class="q-pa-md">
          <div class="text-h6">Historial de acciones:</div>
          <ul>
            <li v-for="(item, index) in state.history" :key="index">
              {{ item }}
            </li>
          </ul>
        </div>
      </q-card>
    </div>

    <!-- Botones adicionales -->
    <div class="row full-width items-center q-my-md">
      <q-btn @click="resetCounter" icon="restart_alt" size="xl" round color="blue" />
      <q-btn
        @click="shareProgress"
        icon="share"
        size="xl"
        round
        color="purple"
      />
    </div>
  </q-page>
</template>

<style scoped>
.q-page {
  max-width: 700px;
  margin: 0 auto;
}
.text-black {
  color: black;
}
</style>
<script setup>
import { reactive, watch } from 'vue';
import { useQuasar } from 'quasar';

// Instancia de Quasar
const $q = useQuasar();

// Estado reactivo
const state = reactive({
  counter: 0,
  history: [],
  maxLimit: 10,
  minLimit: 0,
  motivationalMessage: '',
});

// Mensajes motivadores
const motivationalMessages = [
  "¡Sigue adelante, estás logrando cosas increíbles!",
  "Recuerda: cada paso cuenta.",
  "¡Eres imparable!",
  "El éxito es el resultado de pequeños esfuerzos repetidos.",
  "¡Estás más cerca de tu meta de lo que crees!",
];

// Sincronización con localStorage
const savedData = $q.localStorage.getItem('counterData');
if (savedData) Object.assign(state, savedData);

// Observador para guardar datos automáticamente
watch(state, (newValue) => {
  $q.localStorage.set('counterData', JSON.parse(JSON.stringify(newValue)));
});

// Funciones principales
const increaseCounter = () => {
  if (state.counter < state.maxLimit) {
    state.counter++;
    addToHistory("Incrementó el contador");
    updateMotivationalMessage();
  } else {
    notifyUser("Has alcanzado el límite máximo", "red");
  }
};

const decreaseCounter = () => {
  if (state.counter > state.minLimit) {
    state.counter--;
    addToHistory("Disminuyó el contador");
    updateMotivationalMessage();
  } else {
    notifyUser("Has alcanzado el límite mínimo", "red");
  }
};

const resetCounter = () => {
  state.counter = 0;
  addToHistory("Reinició el contador");
  updateMotivationalMessage();
};

// Historial de acciones
const addToHistory = (action) => {
  state.history.push(`${action} - ${new Date().toLocaleTimeString()}`);
};

// Actualiza el mensaje motivador
const updateMotivationalMessage = () => {
  const randomIndex = Math.floor(Math.random() * motivationalMessages.length);
  state.motivationalMessage = motivationalMessages[randomIndex];
};

// Compartir progreso
const shareProgress = async () => {
  const message = `Progreso del contador: ${state.counter}\nHistorial de acciones:\n${state.history.join(
    "\n"
  )}`;
  try {
    await navigator.share({
      title: "Progreso del contador",
      text: message,
    });
    notifyUser("Progreso compartido exitosamente", "green");
  } catch (err) {
    notifyUser("No se pudo compartir el progreso", "red");
  }
};

// Notificación
const notifyUser = (message, color) => {
  $q.notify({
    message,
    color,
    position: "top",
    timeout: 2000,
  });
};
</script>