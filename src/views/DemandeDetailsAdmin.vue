<template>
  <div class="page-demande-details-admin">
    <!-- Navbar -->
    <header class="navbar">
      <div class="navbar-left">
        <img src="../assets/RobustCodelogowhite.png" alt="Logo RDC" class="logo" />
        <h1>Détails de la Demande</h1>
      </div>
      <div class="navbar-right">
        <router-link to="/demandes" class="nav-btn">Toutes les demandes</router-link>
        <button @click="logout" class="logout-btn">Déconnexion</button>
      </div>
    </header>

    <!-- Contenu principal -->
    <main class="content">
      <section v-if="loading" class="loading">
        Chargement des détails de la demande...
      </section>

      <section v-else-if="!demande" class="empty-state">
        Demande non trouvée.
      </section>

      <section v-else class="demande-details-card">
        <div class="details-header">
          <h2>Demande pour : {{ formatNomComplet(demande.citoyen) }}</h2>
          <span class="demande-statut" :class="getStatutClass(demande.statut)">
            Statut : {{ getStatutLabel(demande.statut) }}
          </span>
        </div>

        <div class="details-body">
          <p><strong>Type de demande :</strong> {{ getTypeDemandeLabel(demande.typeDemande) }}</p>
          <p><strong>Date de soumission :</strong> {{ formatDate(demande.createdAt) }}</p>
          <p v-if="demande.agent">
            <strong>Agent assigné :</strong> {{ formatNomComplet(demande.agent) }}
          </p>
          <div class="commentaires-section">
            <h3>Commentaires de l'agent :</h3>
            <p v-if="demande.commentaires">{{ demande.commentaires }}</p>
            <p v-else>Aucun commentaire.</p>
          </div>
        </div>

        <div class="details-actions">
          <button 
            v-if="demande.statut && demande.statut.nom === 'en traitement'" 
            @click="validateDemande"
            :disabled="isUpdating"
            class="validate-button">
            <span v-if="isUpdating">Validation en cours...</span>
            <span v-else>Valider et signer le document</span>
          </button>

          <a 
            v-if="demande.documentPath"
            :href="`http://localhost:4001/documents/${demande.documentPath.split('/').pop()}`"
            target="_blank"
            class="view-document-link">
            📥 Voir le Document
          </a>
          
          <button @click="goBack" class="back-btn">Retour à la liste</button>
        </div>
      </section>
    </main>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "DemandeDetailsAdmin",
  data() {
    return {
      loading: true,
      isUpdating: false,
      demande: null,
      statutMapping: {
        soumise: "Soumise",
        "en traitement": "En traitement",
        validee: "Validée",
      },
    };
  },
  methods: {
    async fetchDemandeDetails() {
      this.loading = true;
      try {
        const id = this.$route.params.id;
        const token = localStorage.getItem("token");
        const res = await axios.get(`http://localhost:4001/api/demandes/${id}`, {
          headers: { Authorization: `Bearer ${token}` },
        });
        this.demande = res.data;
      } catch (error) {
        console.error("Erreur chargement des détails:", error);
        this.demande = null;
      } finally {
        this.loading = false;
      }
    },
    async validateDemande() {
      this.isUpdating = true;
      try {
        const id = this.$route.params.id;
        const token = localStorage.getItem("token");
        const res = await axios.put(
          `http://localhost:4001/api/demandes/${id}/validate-document`,
          {},
          {
            headers: { Authorization: `Bearer ${token}` },
          }
        );
        alert(res.data.message);
        await this.fetchDemandeDetails();
      } catch (error) {
        console.error("Erreur de validation:", error.response?.data || error.message);
        alert("Erreur lors de la validation et de la signature du document.");
      } finally {
        this.isUpdating = false;
      }
    },
    formatNomComplet(person) {
      if (!person) return "-";
      return [person.nom, person.prenom, person.postnom].filter(Boolean).join(" ");
    },
    getTypeDemandeLabel(type) {
      const mapping = {
        carte_identite: "Carte d'identité",
        acte_naissance: "Acte de naissance",
        acte_mariage: "Acte de mariage",
        acte_residence: "Acte de résidence",
      };
      return mapping[type] || type;
    },
    getStatutLabel(statut) {
      const statutNom = statut && statut.nom ? statut.nom : statut;
      return this.statutMapping[statutNom] || statutNom;
    },
    getStatutClass(statut) {
      const statutNom = statut && statut.nom ? statut.nom : statut;
      const cssClassNom = statutNom ? statutNom.replace(/\s+/g, '_') : '';
      return `statut-${cssClassNom}`;
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString("fr-FR");
    },
    goBack() {
      this.$router.go(-1);
    },
    logout() {
      localStorage.removeItem("token");
      this.$router.push('/');
    },
  },
  mounted() {
    this.fetchDemandeDetails();
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=ABeeZee&family=Inter&family=Ysabeau+Office&display=swap');

.page-demande-details-admin {
  font-family: "ABeeZee", sans-serif;
  background: #f9f9f9;
  min-height: calc(100vh - 70px);
  padding: 30px;
}

/* Navbar */
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #0E2C5A;
  padding: 14px 24px;
  color: white;
  font-family: 'Ysabeau Office', sans-serif;
}
.navbar-left {
  display: flex;
  align-items: center;
  gap: 12px;
}
.logo {
  height: 45px;
}
.navbar-left h1 {
  font-size: 20px;
  font-weight: bold;
}
.navbar-right {
  display: flex;
  align-items: center;
  gap: 12px;
}
.navbar-right .nav-btn {
  background: white;
  color: #0E2C5A;
  padding: 10px 18px;
  border-radius: 10px;
  font-weight: bold;
  font-family: 'Inter', sans-serif;
  text-decoration: none;
  transition: background 0.3s;
}
.navbar-right .nav-btn:hover {
  background: #f0f0f0;
}
.logout-btn {
  background: white;
  color: #0E2C5A;
  padding: 10px 18px;
  border-radius: 10px;
  font-weight: bold;
  font-family: 'Inter', sans-serif;
  border: none;
  cursor: pointer;
  transition: background 0.3s;
}
.logout-btn:hover {
  background: #f0f0f0;
}

/* Contenu */
.demande-details-card {
  background: #fff;
  padding: 24px;
  border-radius: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  margin-top: 20px;
  font-family: "ABeeZee", sans-serif;
}
.details-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}
.details-header h2 {
  font-size: 22px;
  color: #0E2C5A;
  margin: 0;
  font-family: 'Ysabeau Office', sans-serif;
}
.demande-statut {
  font-weight: bold;
  padding: 6px 14px;
  border-radius: 20px;
  color: white;
}
.statut-soumise {
  background-color: #f0ad4e;
}
.statut-en_traitement {
  background-color: #104B71;
}
.statut-validee {
  background-color: #28a745;
}
.details-body {
  margin-bottom: 20px;
  border-top: 1px solid #eee;
  padding-top: 20px;
}
.details-body p {
  margin: 8px 0;
  color: #333;
}
.commentaires-section {
  margin-top: 20px;
  padding: 15px;
  background-color: #f9f9f9;
  border-left: 4px solid #0E2C5A;
  border-radius: 4px;
}

/* Actions */
.details-actions {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}
.details-actions button,
.details-actions .view-document-link {
  padding: 10px 20px;
  border-radius: 10px;
  border: none;
  cursor: pointer;
  font-weight: bold;
  text-decoration: none;
  font-family: "Inter", sans-serif;
  transition: background 0.3s;
}
.validate-button {
  background: #28a745;
  color: white;
}
.validate-button:hover {
  background: #218838;
}
.back-btn {
  background: #6c757d;
  color: white;
}
.back-btn:hover {
  background: #5a6268;
}
.view-document-link {
  background-color: #104B71;
  color: white;
}
.view-document-link:hover {
  background-color: #0E2C5A;
}

/* Messages */
.loading {
  font-size: 18px;
  color: #333;
}
.empty-state {
  font-size: 16px;
  color: #c0392b;
  background: #ffe6e6;
  padding: 12px;
  border-radius: 8px;
  font-weight: bold;
}
</style>