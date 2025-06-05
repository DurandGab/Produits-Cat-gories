<script setup>
import { reactive, onMounted, ref } from "vue";
import { doAjaxRequest } from "@/api";

const currentURL = ref("/api/produits?page=0&size=5&sort=nom,asc")
const previousURL = ref("")
const nextURL = ref("")
const firstURL = ref("")
const lastURL = ref("")
const currentPage = ref(0);

// Pour réinitialiser le formulaire
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

function chargeProduits() {
  // Appel à l'API pour avoir la liste des catégories
  // Trié par code, descendant
  // Verbe HTTP GET par défaut
  doAjaxRequest(currentURL.value)
      .then((json) => {
        data.listeProduits = json._embedded.produits;
        currentURL.value = json._links.self.href;
        nextURL.value = json._links.next?.href || "";
        previousURL.value = json._links.prev?.href || "";
        firstURL.value = json._links.first?.href || "";
        lastURL.value = json._links.last?.href || "";
      })
      .catch(showError);
}

function updateListeProduits(json) {
  data.listeProduits = json._embedded.produits;
  currentURL.value = json._links.self.href;
  nextURL.value = json._links.next?.href || "";
  previousURL.value = json._links.prev?.href || "";
  firstURL.value = json._links.first?.href || "";
  lastURL.value = json._links.last?.href || "";
  const match = currentURL.value.match(/page=(\d+)/);
  currentPage.value = match ? parseInt(match[1], 10) : 0;
}

function goToFirstPage() {
  if (!firstURL.value || currentURL.value === firstURL.value) return;

  doAjaxRequest(firstURL.value)
      .then(updateListeProduits)
      .catch(showError);
}

function goToPreviousPage() {
  if (!previousURL.value || currentURL.value === previousURL.value) return;

  doAjaxRequest(previousURL.value)
      .then(updateListeProduits)
      .catch(showError);
}

function goToNextPage() {
  if (!nextURL.value || currentURL.value === nextURL.value) return;

  doAjaxRequest(nextURL.value)
      .then(updateListeProduits)
      .catch(showError);
}

function goToLastPage() {
  if (!lastURL.value || currentURL.value === lastURL.value) return;

  doAjaxRequest(lastURL.value)
      .then(updateListeProduits)
      .catch(showError);
}

function chargerProduitsParCategorie() {
  if (!categorieSelectionnee.value) return;

  // on ignore la pagination ici
  doAjaxRequest(categorieSelectionnee.value)
      .then((json) => {
        data.listeProduits = json._embedded.produits;
        currentURL.value = categorieSelectionnee.value;
        currentPage.value = 0; // ou aucune pagination si l’endpoint ne le gère pas
      })
      .catch(showError);
}
// A l'affichage du composant, on affiche la liste
onMounted(chargeProduits);

</script>
<template>
  <main>
    <div>
      <h1>Les catégories de produits</h1>
    </div>
    <div>
      <p>Page actuelle : {{ currentPage + 1 }}</p>
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
<!--          <td>-->
<!--            <button @click="deleteEntity(produit._links.self.href)">-->
<!--              Supprimer-->
<!--            </button>-->
<!--          </td>-->
        </tr>
        <tr>
          <td>
            <button @click="goToFirstPage">⇇</button>
          </td>
          <td>
            <button @click="goToPreviousPage">←</button>
          </td>
          <td>
            <button @click="goToNextPage">→</button>
          </td>
          <td>
            <button @click="goToLastPage">⇉</button>
          </td>
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
