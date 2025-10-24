<template>
  <div class="dashboard-bourgmestre">
    <!-- Navbar -->
    <header class="navbar">
      <div class="navbar-left">
        <img src="../assets/RobustCodelogowhite.png" alt="Logo BE" class="logo" />
        <h1>Bourgmestre - Tableau de bord</h1>
      </div>
      <div class="navbar-right">
        <router-link to="/agents" class="nav-btn">Agents</router-link>
        <router-link to="/demandes" class="nav-btn">Demandes</router-link>
        <button class="logout-btn" @click="logout">Déconnexion</button>
      </div>
    </header>

    <!-- Contenu principal -->
    <main class="content">
      <h2 class="content-title">📊 Statistiques de ma commune</h2>

      <section v-if="loading" class="loading">
        Chargement des statistiques...
      </section>

      <section v-else-if="error" class="error">
        ⚠️ {{ error }}
      </section>

      <section v-else class="cards">
        <div class="card">
          <h3>{{ stats.totalDemandes }}</h3>
          <p>Demandes totales</p>
        </div>
        <div class="card">
          <h3>{{ stats.demandesSoumises }}</h3>
          <p>Soumises</p>
        </div>
        <div class="card">
          <h3>{{ stats.demandesEnTraitement }}</h3>
          <p>En traitement</p>
        </div>
        <div class="card">
          <h3>{{ stats.demandesValidees }}</h3>
          <p>Validées</p>
        </div>
        <div class="card">
          <h3>{{ stats.totalAgents }}</h3>
          <p>Agents</p>
        </div>
        <div class="card">
          <h3>{{ stats.totalCitoyens }}</h3>
          <p>Citoyens</p>
        </div>
      </section>
    </main>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "DashboardBourgmestre",
  data() {
    return {
      loading: true,
      error: null,
      stats: {
        totalDemandes: 0,
        demandesSoumises: 0,
        demandesEnTraitement: 0,
        demandesValidees: 0,
        totalAgents: 0,
        totalCitoyens: 0,
      },
    };
  },
  methods: {
    async fetchStats() {
      this.loading = true;
      this.error = null;
      try {
        const token = localStorage.getItem("token");
        const res = await axios.get(
          "https://be-solution-backend.onrender.com/api/dashboard/bourgmestre",
          {
            headers: { Authorization: `Bearer ${token}` },
          }
        );

        // ✅ Mise à jour des statistiques reçues
        this.stats = {
          ...this.stats,
          totalDemandes: res.data.totalDemandes || 0,
          demandesSoumises: res.data.demandesSoumises || 0,
          demandesEnTraitement: res.data.demandesEnTraitement || 0,
          demandesValidees: res.data.demandesValidees || 0,
          totalAgents: res.data.totalAgents || 0,
          totalCitoyens: res.data.totalCitoyens || 0,
        };
      } catch (err) {
        console.error("Erreur chargement stats bourgmestre", err);
        this.error = "Impossible de charger les statistiques.";
      } finally {
        this.loading = false;
      }
    },
    logout() {
      localStorage.removeItem("token");
      window.location.href = "/";
    },
  },
  async mounted() {
    await this.fetchStats();
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=ABeeZee&family=Inter&family=Ysabeau+Office&display=swap');

/* --- NAVBAR --- */
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
  gap: 14px;
}
.nav-btn {
  background: #fff;
  color: #0E2C5A;
  padding: 10px 18px;
  border-radius: 10px;
  font-weight: bold;
  font-family: 'Inter', sans-serif;
  text-decoration: none;
  transition: background 0.3s;
}
.nav-btn:hover {
  background: #f0f0f0;
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

/* --- CONTENU --- */
.content {
  padding: 30px;
  font-family: 'ABeeZee', sans-serif;
  background: #f9f9f9;
  min-height: calc(100vh - 70px);
}
.content-title {
  font-size: 22px;
  font-weight: bold;
  color: #0E2C5A;
  margin-bottom: 24px;
  font-family: 'Ysabeau Office', sans-serif;
}

/* --- CARTES --- */
.cards {
  display: flex;
  gap: 20px;
  flex-wrap: wrap;
  margin-top: 10px;
  justify-content: center;
}
.card {
  background: #fff;
  padding: 22px;
  border-radius: 14px;
  width: 200px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.08);
  text-align: center;
  transition: transform 0.2s;
}
.card:hover {
  transform: translateY(-4px);
}
.card h3 {
  font-size: 28px;
  margin: 0;
  color: #104B71;
  font-family: 'Inter', sans-serif;
}
.card p {
  margin-top: 8px;
  font-size: 15px;
  color: #555;
  font-weight: 500;
}

/* --- ÉTATS --- */
.loading {
  font-size: 18px;
  color: #104B71;
}
.error {
  font-size: 16px;
  color: #c0392b;
  background: #ffe6e6;
  padding: 12px;
  border-radius: 8px;
  font-weight: bold;
}
</style>