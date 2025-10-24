<template>
  <div class="page-agents">
    <!-- Navbar -->
    <header class="navbar">
      <div class="navbar-left">
        <img src="../assets/RobustCodelogowhite.png" alt="Logo BE" class="logo" />
        <h1>Gestion des Agents</h1>
      </div>
      <div class="navbar-right">
        <router-link to="/dashboard-bourgmestre" class="nav-btn">Accueil</router-link>
        <router-link to="/demandes" class="nav-btn">Demandes</router-link>
        <button class="logout-btn" @click="logout">Déconnexion</button>
      </div>
    </header>

    <!-- Liste des agents -->
    <section v-if="loading" class="loading">Chargement des agents...</section>

    <section v-else>
      <h2>Liste des Agents</h2>
      <table>
        <thead>
          <tr>
            <th>Nom complet</th>
            <th>Nom d'utilisateur</th>
            <th>Type de demande</th>
            <th>Action</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="agent in agents" :key="agent.id">
            <td>{{ formatNomComplet(agent) }}</td>
            <td>{{ agent.username }}</td>
            <td>{{ getTypeDemandeLabel(agent.typeDemande) }}</td>
            <td>
              <button @click="deleteAgent(agent.id)">🗑 Supprimer</button>
            </td>
          </tr>
        </tbody>
      </table>
    </section>

    <!-- Formulaire ajout agent -->
    <section class="form-section">
      <h2>Ajouter un nouvel Agent</h2>
      <form @submit.prevent="createAgent">
        <div class="form-group">
          <label>Nom</label>
          <input v-model="form.nom" required />
        </div>
        <div class="form-group">
          <label>Prénom</label>
          <input v-model="form.prenom" required />
        </div>
        <div class="form-group">
          <label>Postnom</label>
          <input v-model="form.postnom" />
        </div>
        <div class="form-group">
          <label>Nom d'utilisateur</label>
          <input v-model="form.username" required />
        </div>
        <div class="form-group">
          <label>Mot de passe</label>
          <input type="password" v-model="form.password" required />
        </div>
        <div class="form-group">
          <label>Type de demande</label>
          <select v-model="form.typeDemande" required>
            <option disabled value="">-- Sélectionner --</option>
            <option value="carte_identite">Carte d'identité</option>
            <option value="acte_naissance">Acte de naissance</option>
            <option value="acte_mariage">Acte de mariage</option>
            <option value="acte_residence">Acte de résidence</option>
          </select>
        </div>
        <button type="submit">Ajouter</button>
      </form>
    </section>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "Agents",
  data() {
    return {
      loading: true,
      agents: [],
      form: {
        nom: "",
        prenom: "",
        postnom: "",
        username: "",
        password: "",
        typeDemande: "",
      },
    };
  },
  methods: {
    async fetchAgents() {
      this.loading = true;
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get(
          "https://be-solution-backend.onrender.com/api/agents",
          { headers: { Authorization: `Bearer ${token}` } }
        );
        this.agents = res.data;
      } catch (error) {
        console.error("Erreur chargement agents", error);
      } finally {
        this.loading = false;
      }
    },
    async createAgent() {
      try {
        const token = localStorage.getItem("token");
        await axios.post(
          "https://be-solution-backend.onrender.com/api/agents",
          this.form,
          { headers: { Authorization: `Bearer ${token}` } }
        );
        alert("Agent ajouté avec succès !");
        this.form = {
          nom: "",
          prenom: "",
          postnom: "",
          username: "",
          password: "",
          typeDemande: "",
        };
        await this.fetchAgents();
      } catch (error) {
        console.error("Erreur ajout agent", error);
        alert("Erreur lors de l'ajout de l'agent.");
      }
    },
    async deleteAgent(id) {
      if (!confirm("Voulez-vous vraiment supprimer cet agent ?")) return;
      try {
        const token = localStorage.getItem("token");
        await axios.delete(
          `https://be-solution-backend.onrender.com/api/agents/${id}`,
          { headers: { Authorization: `Bearer ${token}` } }
        );
        alert("Agent supprimé !");
        await this.fetchAgents();
      } catch (error) {
        console.error("Erreur suppression agent", error);
        alert("Erreur lors de la suppression.");
      }
    },
    formatNomComplet(agent) {
      return [agent.nom, agent.prenom, agent.postnom].filter(Boolean).join(" ");
    },
    getTypeDemandeLabel(value) {
      const mapping = {
        carte_identite: "Carte d'identité",
        acte_naissance: "Acte de naissance",
        acte_mariage: "Acte de mariage",
        acte_residence: "Acte de résidence",
      };
      return mapping[value] || value;
    },
    logout() {
      localStorage.removeItem("token");
      window.location.href = "/";
    },
  },
  async mounted() {
    await this.fetchAgents();
  },
};
</script>

<style scoped>
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
  height: 45px;
}
.navbar-right {
  display: flex;
  gap: 12px;
}
.nav-btn {
  background: white;
  color: #003da5;
  padding: 8px 14px;
  border-radius: 6px;
  font-weight: bold;
  text-decoration: none;
}
.nav-btn:hover {
  background: #f1f1f1;
}
.logout-btn {
  background-color: #FF4C4C;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  font-family: 'Inter', sans-serif;
}
.logout-btn:hover {
  background-color: #e03e3e;
}

/* Table */
table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}
th,
td {
  border: 1px solid #ddd;
  padding: 10px;
}
th {
  background: #0E2C5A;
  color: white;
}

/* Formulaire */
.form-section {
  background: white;
  padding: 20px;
  border-radius: 6px;
  margin-top: 30px;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
  max-width: 600px;
}
.form-group {
  margin-bottom: 16px;
}
label {
  display: block;
  font-weight: 600;
  margin-bottom: 6px;
}
input,
select {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
button {
  background: #0E2C5A;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 6px;
  cursor: pointer;
}
button:hover {
  background: #104B71;
}
</style>