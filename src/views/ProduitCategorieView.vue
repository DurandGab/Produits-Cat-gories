<script setup>
import { reactive, onMounted, ref } from "vue";
import { doAjaxRequest } from "@/api";

const currentURL = ref("/api/produits?page=0&size=5&sort=nom,asc")
const categorieSelectionnee = ref("");

// Pour réinitialiser le formulaire si
const produitVide = {
  nom: "",
  prixUnitaire: "",
  unitesEnStock: "",
  unitesCommandees: "",
};

let data = reactive({
  // Les données saisies dans le formulaire
  formulaireProduit: { ...produitVide },
  // La liste des catégories affichée sous forme de table
  listeProduits: []
});

let dataCategories = reactive({
  listeCategories: []
});

function showError(error) {
  console.log("Erreur : status %d", error.status)
  console.log(error.body);
  alert(error.message);
}

function chargerCategories() {
  doAjaxRequest("/api/categories")
      .then((json) => {
        dataCategories.listeCategories = json._embedded.categories;
      })
      .catch(showError);
}

function chargeProduits() {
  // Appel à l'API pour avoir la liste des catégories
  // Trié par code, descendant
  // Verbe HTTP GET par défaut
  doAjaxRequest("/api/produits?sort=code,desc")
      .then((json) => {
        data.listeProduits = json._embedded.produits;
      })
      .catch(showError);
}

function chargerProduitsParCategorie() {
  if (!categorieSelectionnee.value) return;

  // on ignore la pagination ici
  doAjaxRequest(categorieSelectionnee.value)
      .then((json) => {
        data.listeProduits = json._embedded.produits;
        currentURL.value = categorieSelectionnee.value;
      })
      .catch(showError);
}
// A l'affichage du composant, on affiche la liste
onMounted(chargeProduits);
onMounted(chargerCategories);

</script>
<template>
  <main>
    <div>
      <h1>Les catégories de produits</h1>
      <select v-model="categorieSelectionnee" @change="chargerProduitsParCategorie">
        <option disabled value="">-- Choisissez une catégorie --</option>
        <option v-for="cat in dataCategories.listeCategories" :key="cat.code" :value="cat._links.produits.href">
          {{ cat.libelle }}
        </option>
      </select>
    </div>
    <div>
      <table>
        <caption>Liste des produits</caption>
        <tr>
          <th>Nom</th>
          <th>Prix</th>
          <th>Stock</th>
          <th>Commandés</th>
        </tr>
        <!-- Si le tableau des catégories est vide -->
        <tr v-if="data.listeProduits.length === 0">
          <td colspan="4">Veuillez patienter, chargement des Produits...</td>
        </tr>
        <!-- Si le tableau des catégories n'est pas vide -->
        <tr v-for="produit in data.listeProduits" :key="produit.code">
          <td>{{ produit.nom }}</td>
          <td>{{ produit.prixUnitaire }}</td>
          <td>{{ produit.unitesEnStock }}</td>
          <td>{{ produit.unitesCommandees }}</td>
        </tr>
      </table>
    </div>
  </main>
</template>




<style scoped>
td,
th {
  border: 1px solid #ddd;
  padding: 8px;
}

th {
  padding-top: 12px;
  padding-bottom: 12px;
  text-align: left;
  background-color: #232623;
  color: rgb(255, 255, 255);
}
</style>
