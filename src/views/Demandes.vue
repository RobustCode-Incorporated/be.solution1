<template>
  <div class="page-demandes">
    <!-- Navbar -->
    <header class="navbar">
      <div class="navbar-left">
        <img src="../assets/RobustCodelogowhite.png" alt="Logo BE" class="logo" />
        <h1>Gestion des Demandes</h1>
      </div>
      <div class="navbar-right">
        <router-link to="/dashboard-bourgmestre" class="nav-btn">Dashboard</router-link>
        <router-link to="/agents" class="nav-btn">Agents</router-link>
        <button class="logout-btn" @click="logout">Déconnexion</button>
      </div>
    </header>

    <!-- Filtres -->
    <section class="filters">
      <label for="statut">Filtrer par statut :</label>
      <select v-model="filtreStatut" @change="fetchDemandes">
        <option value="">Toutes</option>
        <option value="soumise">Soumise</option>
        <option value="en_traitement">En traitement</option>
        <option value="validee">Validée</option>
      </select>
    </section>

    <!-- Chargement / erreurs -->
    <section v-if="loading" class="loading">⏳ Chargement des demandes...</section>

    <!-- Tableau -->
    <section v-else>
      <table>
        <thead>
          <tr>
            <th>Type de demande</th>
            <th>Citoyen</th>
            <th>Date</th>
            <th>Statut</th>
            <th>Document</th>
            <th>Actions</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="demande in demandes" :key="demande.id">
            <td>{{ getTypeDemandeLabel(demande.typeDemande) }}</td>
            <td>{{ formatNomComplet(demande.citoyen) }}</td>
            <td>{{ formatDate(demande.createdAt) }}</td>
            <td>
              <span :class="['badge', getBadgeClass(demande.statut?.nom)]">
                {{ getStatutLabel(demande.statut) }}
              </span>
            </td>
            <td>
              <a v-if="demande.documentPath"
                 :href="`https://be-solution-backend.onrender.com/documents/${demande.documentPath}`"
                 target="_blank"
                 class="document-link-cell">
                📥 Voir
              </a>
              <span v-else>N/A</span>
            </td>
            <td>
              <button
                v-if="demande.statut && demande.statut.nom === 'en_traitement'"
                @click="validateDemande(demande.id)"
                class="validate-btn"
              >
                ✅ Valider et Signer
              </button>
              <button
                v-else
                @click="goToDemandeDetails(demande.id)"
                class="view-btn"
              >
                🔎 Voir la demande
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </section>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "Demandes",
  data() {
    return {
      loading: true,
      demandes: [],
      filtreStatut: "",
      statutMapping: {
        soumise: "Soumise",
        en_traitement: "En traitement",
        validee: "Validée",
      },
    };
  },
  methods: {
    async fetchDemandes() {
      this.loading = true;
      try {
        const token = localStorage.getItem("token");
        let url = "https://be-solution-backend.onrender.com/api/demandes";
        if (this.filtreStatut) {
          url += `?statut=${this.filtreStatut}`;
        }
        const res = await axios.get(url, {
          headers: { Authorization: `Bearer ${token}` },
        });
        this.demandes = res.data;
      } catch (error) {
        console.error("Erreur chargement demandes", error);
      } finally {
        this.loading = false;
      }
    },
    async validateDemande(id) {
      if (confirm("Êtes-vous sûr de vouloir valider et signer ce document ?")) {
        try {
          const token = localStorage.getItem("token");
          await axios.put(
            `https://be-solution-backend.onrender.com/api/demandes/${id}/validate-document`,
            {},
            { headers: { Authorization: `Bearer ${token}` } }
          );
          alert("✅ Demande validée et document signé avec succès !");
          this.fetchDemandes();
        } catch (error) {
          console.error("Erreur de validation:", error.response?.data);
          alert("⚠️ Erreur lors de la validation. Le document doit être 'en traitement'.");
        }
      }
    },
    formatNomComplet(person) {
      if (!person) return "-";
      return [person.nom, person.postnom, person.prenom].filter(Boolean).join(" ");
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
    getBadgeClass(statutNom) {
      if (!statutNom) return "";
      switch (statutNom) {
        case "soumise": return "badge-blue";
        case "en_traitement": return "badge-orange";
        case "validee": return "badge-green";
        default: return "badge-gray";
      }
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString("fr-FR");
    },
    goToDemandeDetails(id) {
      this.$router.push({ name: "DemandeDetailsAdmin", params: { id } });
    },
    logout() {
      localStorage.removeItem("token");
      this.$router.push("/");
    },
  },
  mounted() {
    this.fetchDemandes();
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=ABeeZee&family=Inter&family=Ysabeau+Office&display=swap');

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
  gap: 10px;
}
.logo {
  height: 40px;
}
.navbar-right {
  display: flex;
  gap: 12px;
}
.nav-btn {
  background: white;
  color: #0E2C5A;
  padding: 8px 14px;
  border-radius: 6px;
  font-weight: bold;
  text-decoration: none;
}
.nav-btn:hover {
  background: #f1f1f1;
}

/* Filtres */
.filters {
  margin: 20px 0;
  font-family: "Inter", sans-serif;
}
.filters label {
  font-weight: bold;
  margin-right: 10px;
  color: #0E2C5A;
}
.filters select {
  padding: 6px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

/* Table */
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 10px;
  font-family: "ABeeZee", sans-serif;
}
th,
td {
  border: 1px solid #ddd;
  padding: 10px;
  text-align: center;
}
th {
  background: #0E2C5A;
  color: white;
  text-transform: uppercase;
  font-size: 14px;
}

/* Boutons */
button {
  background: #0E2C5A;
  color: white;
  padding: 6px 12px;
  border-radius: 5px;
  border: none;
  cursor: pointer;
}
button:hover {
  background: #104B71;
}
.validate-btn {
  background: #28a745;
}
.validate-btn:hover {
  background: #218838;
}
.view-btn {
  background: #0E2C5A;
}
.view-btn:hover {
  background: #104B71;
}

/* Badges Statuts */
.badge {
  padding: 4px 10px;
  border-radius: 12px;
  font-weight: bold;
  font-size: 13px;
  color: white;
}
.badge-blue {
  background: #007bff;
}
.badge-orange {
  background: #ff9800;
}
.badge-green {
  background: #28a745;
}
.badge-gray {
  background: #6c757d;
}

/* Document link */
.document-link-cell {
  background: #104B71;
  color: white;
  padding: 5px 8px;
  border-radius: 4px;
  text-decoration: none;
}
.document-link-cell:hover {
  background: #0E2C5A;
}

/* Logout */
.logout-btn {
  background-color: #FF4C4C;
  color: white;
  padding: 8px 14px;
  border-radius: 6px;
  border: none;
  font-weight: bold;
  cursor: pointer;
  font-family: 'Inter', sans-serif;
}
.logout-btn:hover {
  background-color: #e03e3e;
}
</style>