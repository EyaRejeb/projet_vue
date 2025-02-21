<template>
  <div class="form-container">
    <h2>Enregistrer une participation</h2>

    <!-- Sélection de la personne -->
    <div class="form-group">
      <label>Personne</label>
      <select v-model="selectedPers">
        <option :value="null" disabled>-- Choisir une personne --</option>
        <option
          v-for="pers in personnes"
          :key="pers.matricule"
          :value="pers.matricule"
        >
          {{ pers.nom }} {{ pers.prenom }}
        </option>
      </select>
    </div>

    <!-- Sélection du projet -->
    <div class="form-group">
      <label>Projet</label>
      <select v-model="selectedProjet">
        <option :value="null" disabled>-- Choisir un projet --</option>
        <option
          v-for="proj in projets"
          :key="proj.id"
          :value="proj.id"
        >
          {{ proj.nom }}
        </option>
      </select>
    </div>

    <!-- Rôle -->
    <div class="form-group">
      <label>Rôle</label>
      <input type="text" v-model="inputRole" placeholder="Entrez le rôle" />
    </div>

    <!-- Pourcentage (slider) -->
    <div class="form-group">
      <label>Pourcentage</label>
      <div class="slider-container">
        <input
          type="range"
          min="0"
          max="100"
          step="1"
          v-model="selectedPourcentage"
        />
        <span class="slider-value">{{ selectedPourcentage }}%</span>
      </div>
    </div>

    <!-- Bouton Valider -->
    <button class="btn-submit" @click="enregistrerParticipation">Enregistrer</button>

    <!-- Affichage de l'erreur sous forme de toast -->
    <div v-if="toastMessage" class="toast" :class="toastClass">
      {{ toastMessage }}
    </div>
  </div>
</template>

<script setup>
import { onMounted, reactive, toRefs } from "vue";
import doAjaxRequest from "@/util/util.js";

// Les données du composant
let data = reactive({
  projets: [],
  personnes: [],
  inputRole: '',
  selectedProjet: null,
  selectedPers: null,
  selectedPourcentage: 0,
  toastMessage: '',   // New toastMessage
  toastClass: ''      // New toastClass for dynamic class binding (success or error)
});

const {
  personnes,
  projets,
  selectedPers,
  selectedProjet,
  inputRole,
  selectedPourcentage,
  toastMessage,
  toastClass
} = toRefs(data)

// Enregistrer la participation
function enregistrerParticipation() {
  data.toastMessage = '';

  if (!data.selectedPers || !data.selectedProjet) {
    showToast('Sélectionnez une personne et un projet.', 'error');
    return;
  }
  if (!data.inputRole) {
    showToast('Précisez le rôle.', 'error');
    return;
  }
  if (data.selectedPourcentage <= 0) {
    showToast('Le pourcentage doit être > 0.', 'error');
    return;
  }

  const url =
    `http://localhost:8989/api/gestion/participation?matricule=${data.selectedPers}` +
    `&codeProjet=${data.selectedProjet}` +
    `&role=${encodeURIComponent(data.inputRole)}` +
    `&pourcentage=${Number(data.selectedPourcentage/100)}`;

  doAjaxRequest(url, { method: 'POST' })
    .then((result) => {
      console.log('Réponse du serveur :', result);
      showToast("Participation enregistrée avec succès !", 'success');
      data.selectedPers = null;
      data.selectedProjet = null;
      data.inputRole = '';
      data.selectedPourcentage = 0;
    })
    .catch(error => showToast(error.message, 'error'));
}

function fetchProjet() {
  doAjaxRequest("/api/projets")
    .then((result) => {
      data.projets = result._embedded.projets;
    })
    .catch(error => showToast(error.message, 'error'));
}

function fetchPersonne() {
  doAjaxRequest("/api/personnes")
    .then((result) => {
      data.personnes = result._embedded.personnes;
    })
    .catch(error => showToast(error.message, 'error'));
}

// Toast function with dynamic classes for success/error
function showToast(message, type) {
  data.toastMessage = message;
  data.toastClass = type === 'success' ? 'toast-success' : 'toast-error';

  setTimeout(() => {
    data.toastMessage = ''; // Hide the toast after 3 seconds
  }, 3000);
}

onMounted(() => {
  fetchProjet();
  fetchPersonne();
});
</script>

<style scoped>
/* Container styles */
.form-container {
  max-width: 420px;
  margin: 2rem auto;
  padding: 2.5rem;
  background-color: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

/* Title */
.form-container h2 {
  margin-bottom: 1.8rem;
  font-size: 1.5rem;
  font-weight: 600;
  color: #4F9F6A; /* Soft green color */
  text-align: center;
}

/* Form Group */
.form-group {
  margin-bottom: 1.6rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.6rem;
  font-weight: 600;
  color: #333;
}

.form-group select,
.form-group input[type="text"] {
  width: 100%;
  padding: 0.8rem;
  font-size: 1rem;
  border: 1px solid #ddd;
  border-radius: 8px;
  background-color: #fafafa;
  transition: border-color 0.3s ease;
}

.form-group select:focus,
.form-group input[type="text"]:focus {
  border-color: #4F9F6A;
  outline: none;
}

/* Slider styles */
.slider-container {
  display: flex;
  align-items: center;
  gap: 1.2rem;
}

.slider-container input[type="range"] {
  flex: 1;
  background-color: #e6e6e6;
  border-radius: 8px;
  height: 8px;
}

.slider-container input[type="range"]:focus {
  background-color: #A4D8A7;
}

/* Slider value display */
.slider-value {
  font-weight: 500;
  color: #333;
}

/* Submit Button */
.btn-submit {
  display: inline-block;
  padding: 0.8rem 1.4rem;
  font-size: 1rem;
  color: #fff;
  background-color: #4F9F6A;
  border-radius: 8px;
  border: none;
  cursor: pointer;
  transition: background-color 0.3s ease, transform 0.2s ease;
  width: 100%;
}

.btn-submit:hover {
  background-color: #3D8C5F;
  transform: scale(1.05);
}

/* Toast Message */
.toast {
  position: absolute;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 0.8rem 1.2rem;
  font-size: 1rem;
  border-radius: 8px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  color: #fff;
  z-index: 999;
  width: 80%;
  opacity: 0;
  transition: opacity 0.5s ease;
}

.toast-success {
  background-color: #B0E57C;
  opacity: 1;
}

.toast-error {
  background-color: #f6d688;
  opacity: 1;
}
</style>
