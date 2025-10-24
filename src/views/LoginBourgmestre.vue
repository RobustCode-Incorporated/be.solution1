<template>
  <div class="login-container">
    <!-- Bloc gauche -->
    <div class="left-panel">
      <img src="../assets/RobustCodelogowhite.png" alt="Logo Robust Code" class="logo" />
      <h1 class="title">ROBUST ADMINISTRATIVE SYSTEM</h1>
    </div>

    <!-- Bloc droit -->
    <div class="right-panel">
      <h1 class="form-title">Connexion Bourgmestre</h1>
      <form class="login-form" @submit.prevent="handleLogin">
        <div class="form-group">
          <label>Nom d'utilisateur</label>
          <input v-model="username" type="text" placeholder="Nom d'utilisateur" required />
        </div>

        <div class="form-group">
          <label>Mot de passe</label>
          <input type="password" v-model="password" placeholder="Mot de passe" required />
        </div>

        <button type="submit" :disabled="loading">
          {{ loading ? "Connexion..." : "SE CONNECTER" }}
        </button>
      </form>

      <p v-if="error" class="error">{{ error }}</p>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "LoginBourgmestre",
  data() {
    return {
      username: "",
      password: "",
      loading: false,
      error: null,
    };
  },
  methods: {
    async handleLogin() {
      this.loading = true;
      this.error = null;
      try {
        const res = await axios.post(
          "https://be-solution-backend.onrender.com/api/administrateurs/login",
          {
            username: this.username,
            password: this.password,
          }
        );

        // Sauvegarder le token dans le stockage local
        localStorage.setItem("token", res.data.token);

        // Redirection vers le tableau de bord
        this.$router.push("/dashboard-bourgmestre");
      } catch (err) {
        console.error("Erreur connexion bourgmestre :", err);
        this.error = err.response?.data?.message || "Connexion échouée. Vérifiez vos identifiants.";
      } finally {
        this.loading = false;
      }
    },
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=ABeeZee&family=Inter&family=Ysabeau+Office&display=swap');

.login-container {
  display: flex;
  height: 100vh;
  font-family: 'ABeeZee', sans-serif;
}

/* Bloc gauche (branding) */
.left-panel {
  width: 704px;
  height: 100vh;
  background-color: #0E2C5A;
  color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
}

.logo {
  width: 280px;
  margin-bottom: 30px;
}

.title {
  font-family: 'Ysabeau Office', sans-serif;
  font-size: 1.8rem;
  letter-spacing: 1.5px;
  font-weight: bold;
}

/* Bloc droit (formulaire) */
.right-panel {
  flex: 1;
  background: #fff;
  display: flex;
  flex-direction: column;
  justify-content: center;
  padding: 0 80px;
}

.form-title {
  font-size: 24px;
  font-weight: bold;
  margin-bottom: 28px;
  color: #0E2C5A;
  text-align: center;
  font-family: 'Ysabeau Office', sans-serif;
}

.form-group {
  margin-bottom: 20px;
}

.login-form label {
  font-weight: 600;
  margin-bottom: 6px;
  display: block;
  color: #333;
}

.login-form input {
  width: 100%;
  max-width: 437px;
  height: 50px;
  padding: 12px;
  border: 1px solid #ccc;
  border-radius: 10px;
  font-size: 15px;
  transition: border-color 0.3s;
}

.login-form input:focus {
  border-color: #104B71;
  outline: none;
}

.login-form button {
  background-color: #104B71;
  color: white;
  font-family: 'Inter', sans-serif;
  font-size: 16px;
  border: none;
  padding: 14px;
  border-radius: 12px;
  cursor: pointer;
  width: 100%;
  max-width: 437px;
  height: 60px;
  font-weight: bold;
  letter-spacing: 1px;
  transition: background-color 0.3s;
}

.login-form button:disabled {
  background-color: gray;
  cursor: not-allowed;
}

.login-form button:hover:not(:disabled) {
  background-color: #0d3c5a;
}

.error {
  color: red;
  margin-top: 15px;
  text-align: center;
  font-weight: bold;
  font-family: 'Inter', sans-serif;
}
</style>