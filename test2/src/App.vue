<template>
  <header>
   

    <div class="wrapper">
      <HelloWorld msg="CRUD " />
    </div>
  </header>

  <main>
    <!-- Bouton pour ajouter un nouvel élément -->
    <button @click="addItemModal">Ajouter</button>

    <!-- Champ d'entrée de recherche -->
    <input type="text" id="searchInput" v-model="searchQuery" @input="searchItems(searchQuery)" placeholder="Rechercher ....." />

    <!-- Tableau -->  
    <table>
      <thead style="background-color: green;">
        <tr>
          <th>Id</th>
          <th>Name</th>
          <th>Email</th>
          <th>Devise</th>
          <th>Action</th> <!-- Nouvelle colonne pour les boutons Modifier et Supprimer -->
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in filteredItems" :key="item.id">
          <td>{{ item.id }}</td>
          <td>{{ item.name }}</td>
          <td>{{ item.email }}</td>
          <td>{{ item.devise }}</td>
          <td>
            <button @click="openEditModal(item)" class="edit-btn">Modifier</button>
            <button @click="deleteItem(item.id)" class="delete-btn">Supprimer</button>
          </td>
        </tr>
      </tbody>
    </table>
  </main>

  <!-- Modal pour l'ajout/édition -->
  <div id="modal" class="modal" v-show="showModal">
    <div class="modal-content">
      <h2>Ajouter un nouvel élément</h2>
      <div class="inputAjout">
        <input type="text" v-model="newItem.name" placeholder="Nom">
        <input type="text" v-model="newItem.email" placeholder="Email">
        <input type="text" v-model="newItem.devise" placeholder="Devise">
      </div>
      <button @click="addItemAndShowMessage">Ajouter</button>
      <button @click="closeModal">Annuler</button>
    </div>
  </div>

  <!-- Modal pour la modification -->
  <div id="edit-modal" class="modal" v-show="showEditModal">
    <div class="modal-content">
      <h2>Modifier l'élément</h2>
      <div class="inputAjout">
        <input type="text" v-model="editedItem.name" placeholder="Nom">
        <input type="text" v-model="editedItem.email" placeholder="Email">
        <input type="text" v-model="editedItem.devise" placeholder="Devise">
      </div>
      <button @click="confirmEdit">Confirmer</button>
      <button @click="cancelEdit">Annuler</button>
    </div>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld.vue'; // Assurez-vous d'importer correctement le composant HelloWorld

export default {
  components: {
    HelloWorld,
  },
  data() {
    return {
      items: [], // Tableau pour stocker les données
      filteredItems: [], // Tableau pour stocker les éléments filtrés
      showModal: false, // Indicateur pour afficher ou masquer le modal
      newItem: { name: '', email: '', devise: '' }, // Objet pour stocker les détails du nouvel élément
      searchQuery: '', // Terme de recherche
      showEditModal: false,
      editedItem: {
        id: null,
        name: '',
        email: '',
        devise: ''
      }
    };
  },
  methods: {
    // Fonction pour charger les données initiales depuis le fichier JSON ou le stockage local
    loadInitialData() {
      let jsonData = localStorage.getItem('crudData');
      if (!jsonData) {
        // Si aucune donnée n'est présente dans le stockage local, utilisez les données initiales du fichier JSON
        fetch('data.json')
          .then(response => response.json())
          .then(jsonData => {
            this.items = jsonData;
            localStorage.setItem('crudData', JSON.stringify(jsonData));
            this.filteredItems = jsonData; // Mettre à jour les éléments filtrés avec toutes les données
          })
          .catch(error => console.error('Erreur lors du chargement des données:', error));
      } else {
        // Si des données sont déjà présentes dans le stockage local, utilisez-les
        this.items = JSON.parse(jsonData);
        this.filteredItems = JSON.parse(jsonData); // Mettre à jour les éléments filtrés avec toutes les données
      }
    },
    // Fonction pour afficher le modal d'ajout
    addItemModal() {
      this.showModal = true; // Afficher le modal
    },
    // Fonction pour ajouter un nouvel élément et afficher le message de succès
    addItemAndShowMessage() {
      if (!this.newItem.name || !this.newItem.email || !this.newItem.devise) {
        alert("Veuillez remplir tous les champs.");
        return;
      }

      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(this.newItem.email)) {
        alert("Veuillez entrer une adresse e-mail valide.");
        return;
      }

      // Trouver l'ID le plus élevé parmi les éléments existants
      let maxId = 0;
      this.items.forEach(item => {
        if (item.id > maxId) {
          maxId = item.id;
        }
      });

      // Incrémenter l'ID le plus élevé de 1 pour obtenir le nouvel ID
      const newId = maxId + 1;

      const newItem = {
        id: newId,
        name: this.newItem.name,
        email: this.newItem.email,
        devise: this.newItem.devise
      };
      this.items.push(newItem);
      localStorage.setItem('crudData', JSON.stringify(this.items));
      this.newItem = { name: '', email: '', devise: '' }; // Réinitialiser les champs du formulaire
      this.showModal = false; // Masquer le modal
      alert("Ajout réussi !");
      this.filteredItems = this.items; // Mettre à jour les éléments filtrés avec toutes les données
    },

    // Fonction pour fermer le modal
    closeModal() {
      this.showModal = false; // Masquer le modal
      this.newItem = { name: '', email: '', devise: '' }; // Réinitialiser les champs du formulaire
    },

    // Fonction pour supprimer un élément
    deleteItem(itemId) {
      // Boîte de dialogue de confirmation
      if (confirm("Êtes-vous sûr de vouloir supprimer cet élément?")) {
        // Filtrer les données pour supprimer l'élément avec l'ID correspondant
        this.items = this.items.filter(item => item.id !== itemId);
        // Mettre à jour les données dans le stockage local
        localStorage.setItem('crudData', JSON.stringify(this.items));
        this.filteredItems = this.items; // Mettre à jour les éléments filtrés avec les données mises à jour
      }
    },

    // Fonction pour rechercher des éléments
    searchItems(keyword) {
      this.filteredItems = this.items.filter(item =>
        item.name.toLowerCase().includes(keyword.toLowerCase()) ||
        item.email.toLowerCase().includes(keyword.toLowerCase()) ||
        item.devise.toLowerCase().includes(keyword.toLowerCase())
      );
    },

    // Fonction pour ouvrir le modal de modification et remplir les champs avec les données de l'élément sélectionné
    openEditModal(item) {
      this.showEditModal = true; // Afficher le modal de modification
      this.editedItem.id = item.id; // Remplir l'ID de l'élément sélectionné
      this.editedItem.name = item.name; // Remplir le champ "name" avec la valeur de l'élément sélectionné
      this.editedItem.email = item.email; // Remplir le champ "email" avec la valeur de l'élément sélectionné
      this.editedItem.devise = item.devise; // Remplir le champ "devise" avec la valeur de l'élément sélectionné
    },

      // Fonction pour confirmer la modification
    confirmEdit() {
      // Trouver l'index de l'élément à modifier
      const index = this.items.findIndex(item => item.id === this.editedItem.id);
      
      // Vérifier si l'index est valide
      if (index !== -1) {
        // Mettre à jour l'élément dans le tableau des éléments
        this.items[index].name = this.editedItem.name;
        this.items[index].email = this.editedItem.email;
        this.items[index].devise = this.editedItem.devise;
        
        // Mettre à jour les données dans le stockage local
        localStorage.setItem('crudData', JSON.stringify(this.items));

        // Mettre à jour les éléments filtrés avec les données mises à jour
        this.filteredItems = [...this.items];
      }

      // Cacher le modal de modification
      this.showEditModal = false;

      // Réinitialiser l'élément édité
      this.editedItem = { id: null, name: '', email: '', devise: '' };
    },

    // Fonction pour annuler la modification
    cancelEdit() {
      // Cacher le modal de modification
      this.showEditModal = false;

      // Réinitialiser l'élément édité
      this.editedItem = { id: null, name: '', email: '', devise: '' };
    },
  },
  created() {
    this.loadInitialData(); // Charger les données initiales lors de la création du composant
  },
};
</script>
