<template>
  <div class="login-bg">
    <div class="login-main">
      <div class="login-card">
        <!-- Left: Login Form -->
        <div class="login-form-col">
          <div class="login-logo-row">
            <!-- <img src="/Designer.png" alt="PawFect" class="login-logo-img" /> -->
            <span class="login-logo-text">PAWFECT</span>
          </div>
          <h1 class="login-title">Log in to your Account</h1>
          <p class="login-sub">Welcome back! Please enter your details.</p>
          <form class="login-main-form" @submit.prevent="handleLogin">
            <div class="login-input-group">
              <span class="login-input-icon">
                <svg width="20" height="20" fill="none" stroke="#f7871f" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M4 4h12v12H4z" opacity="0"/><path d="M4 4h12v12H4z"/><path d="M4 4l8 6 8-6"/></svg>
              </span>
              <input type="email" v-model="loginEmail" placeholder="Email" required />
            </div>
            <div class="login-input-group">
              <span class="login-input-icon">
                <svg width="20" height="20" fill="none" stroke="#f7871f" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="3" y="8" width="14" height="9" rx="2"/><path d="M7 8V6a5 5 0 0 1 10 0v2"/></svg>
              </span>
              <input :type="showPassword ? 'text' : 'password'" v-model="loginPassword" placeholder="Password" required />
              <span class="login-eye" @click="showPassword = !showPassword">
                <svg v-if="!showPassword" width="20" height="20" fill="none" stroke="#888" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M1 10s4-6 9-6 9 6 9 6-4 6-9 6-9-6-9-6z"/><circle cx="10" cy="10" r="3"/></svg>
                <svg v-else width="20" height="20" fill="none" stroke="#f7871f" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M17.94 17.94A10.94 10.94 0 0 1 10 19C5.58 19 1.73 15.36 1.05 10.94M23 1L1 23"/><path d="M9.53 9.53A3 3 0 0 0 13 13"/></svg>
              </span>
            </div>
            <div class="login-options-row">
              <label class="login-remember">
                <input type="checkbox" v-model="rememberMe" />
                <span class="checkmark"></span>
                Remember me
              </label>
              <a href="#" class="login-link">Forgot Password?</a>
            </div>
            <button
              class="login-btn"
              type="submit"
              :disabled="isLoading"
            >
              <span v-if="isLoading" class="spinner"></span>
              {{ isLoading ? 'Logging in...' : 'Log in' }}
            </button>
          </form>
          <div v-if="errorMessage" class="error-message">
            {{ errorMessage }}
          </div>
          <div class="login-bottom-row">
            <span>Don't have an account?</span>
            <a href="/signup" class="login-link">Create an account</a>
          </div>
        </div>
        <!-- Right: Illustration -->
        <div class="login-illustration-col">
          <div class="login-illustration-bg">
            <!-- <img src="https://cdn.dribbble.com/users/1615584/screenshots/17332813/media/2e2e2e2e2e2e2e2e2e2e2e2e2e2e2e2e2e2e2e2e.png" alt="Dashboard Illustration" class="login-illustration-img" /> -->
            <div class="login-illustration-text">
              <h2>Connect with every application.</h2>
              <p>Your pawfect place for adoption</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'LoginPage',
  data() {
    return {
      loginEmail: '',
      loginPassword: '',
      rememberMe: false,
      showPassword: false,
      isLoading: false,
      errorMessage: '',
    };
  },
  methods: {
    async handleLogin() {
      if (!this.loginEmail || !this.loginPassword) {
        this.errorMessage = 'Email and password are required';
        return;
      }
      const emailPattern = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
      if (!emailPattern.test(this.loginEmail)) {
        this.errorMessage = 'Please enter a valid email address';
        return;
      }

      this.isLoading = true;
      this.errorMessage = '';

      try {
        const response = await axios.post('http://localhost/Pawfect-final--main/backend/login.php', {
          email: this.loginEmail,
          password: this.loginPassword
        });

        const extractedData = response.data;

        if (extractedData && extractedData.success === true) {
          if (extractedData.token) {
            localStorage.setItem('token', extractedData.token);
          }

          if (extractedData.user) {
            localStorage.setItem('user', JSON.stringify(extractedData.user));
          }

          if (this.rememberMe) {
            localStorage.setItem('rememberedEmail', this.loginEmail);
          } else {
            localStorage.removeItem('rememberedEmail');
          }

          this.$nextTick(() => {
            const targetRoute = extractedData.user && extractedData.user.role === 'admin'
              ? '/admin'
              : '/home';

            this.$router.push(targetRoute).catch(err => {
              if (err.name !== 'NavigationDuplicated') {
                console.error('Navigation error:', err);
              }
            });
          });
        } else {
          this.errorMessage = (extractedData && (extractedData.message || extractedData.error)) || 'Login failed. Please check your credentials.';
        }
      } catch (error) {
        this.errorMessage = 'An error occurred during login. Please try again.';
      } finally {
        this.isLoading = false;
      }
    },
  },
  mounted() {
    const rememberedEmail = localStorage.getItem('rememberedEmail');
    if (rememberedEmail) {
      this.loginEmail = rememberedEmail;
      this.rememberMe = true;
    }
  }
};
</script>

<style scoped>
.login-bg {
  min-height: 100vh;
  background: #f4f6fa;
  display: flex;
  align-items: center;
  justify-content: center;
}
.login-main {
  width: 100vw;
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
}
.login-card {
  display: flex;
  background: #fff;
  border-radius: 24px;
  box-shadow: 0 8px 40px rgba(0,0,0,0.10);
  overflow: hidden;
  max-width: 950px;
  width: 100%;
  min-height: 600px;
}
.login-form-col {
  flex: 1.1;
  padding: 56px 48px 48px 48px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  background: #fff;
}
.login-logo-row {
  display: flex;
  align-items: center;
  gap: 0.7rem;
  margin-bottom: 1.5rem;
}
.login-logo-img {
  width: 38px;
  height: 38px;
}
.login-logo-text {
  font-size: 1.5rem;
  font-weight: 800;
  color: #f7871f;
  letter-spacing: 1px;
}
.login-title {
  font-size: 2.1rem;
  font-weight: 700;
  margin-bottom: 0.3rem;
  color: #222;
  letter-spacing: 1px;
}
.login-sub {
  color: #888;
  font-size: 1.05rem;
  margin-bottom: 2.2rem;
}
.login-main-form {
  width: 100%;
  display: flex;
  flex-direction: column;
  gap: 1.1rem;
}
.login-input-group {
  width: 100%;
  display: flex;
  align-items: center;
  background: #f7f8fa;
  border-radius: 8px;
  border: 1.5px solid #e0e0e0;
  margin-bottom: 0.2rem;
  position: relative;
}
.login-input-icon {
  margin-left: 1rem;
  color: #f7871f;
  display: flex;
  align-items: center;
}
.login-input-group input {
  flex: 1;
  border: none;
  background: transparent;
  padding: 0.95rem 1rem 0.95rem 0.5rem;
  font-size: 1rem;
  outline: none;
  color: #222;
}
.login-eye {
  margin-right: 1rem;
  cursor: pointer;
  display: flex;
  align-items: center;
}
.login-options-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0.7rem;
  margin-top: 0.2rem;
}
.login-remember {
  display: flex;
  align-items: center;
  font-size: 0.97rem;
  color: #555;
  cursor: pointer;
  position: relative;
}
.login-remember input[type="checkbox"] {
  accent-color: #f7871f;
  margin-right: 0.5rem;
}
.login-link {
  color: #f7871f;
  text-decoration: none;
  font-size: 0.97rem;
  font-weight: 500;
  transition: color 0.2s;
}
.login-link:hover {
  color: #ff9800;
  text-decoration: underline;
}
.login-btn {
  width: 100%;
  padding: 0.95rem 0;
  background: #e99a08;
  color: #fff;
  border: none;
  border-radius: 8px;
  font-size: 1.1rem;
  font-weight: 700;
  margin-top: 0.7rem;
  cursor: pointer;
  transition: background 0.2s;
  position: relative;
}
.login-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}
.login-btn .spinner {
  position: absolute;
  right: 1rem;
  top: 50%;
  transform: translateY(-50%);
  width: 20px;
  height: 20px;
  border: 2px solid #fff;
  border-top-color: transparent;
  border-radius: 50%;
  animation: spin 1s linear infinite;
}
.login-btn:hover {
  background: #f7871f;
  color: #fff;
}
.login-bottom-row {
  margin-top: 1.5rem;
  text-align: center;
  color: #888;
  font-size: 0.97rem;
}
.login-bottom-row .login-link {
  margin-left: 0.3rem;
  font-weight: 600;
}
.login-illustration-col {
  flex: 1.2;
  background: #fc9d04;
  display: flex;
  align-items: center;
  justify-content: center;
  position: relative;
}
.login-illustration-bg {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  position: relative;
}
.login-illustration-img {
  width: 320px;
  height: 220px;
  object-fit: contain;
  margin-bottom: 2.5rem;
  margin-top: 2.5rem;
}
.login-illustration-text {
  color: #fff;
  text-align: center;
  max-width: 340px;
}
.login-illustration-text h2 {
  font-size: 1.3rem;
  font-weight: 700;
  margin-bottom: 0.7rem;
}
.login-illustration-text p {
  font-size: 1.05rem;
  color: #e0e0e0;
}
.error-message {
  color: #ff5252;
  font-size: 0.9rem;
  margin-top: 0.5rem;
  text-align: center;
}
@media (max-width: 1100px) {
  .login-card {
    flex-direction: column;
    min-height: 0;
    max-width: 500px;
  }
  .login-illustration-col {
    display: none;
  }
  .login-form-col {
    padding: 40px 24px 32px 24px;
  }
}
@media (max-width: 600px) {
  .login-main {
    padding: 0 0.5rem;
  }
  .login-form-col {
    padding: 24px 6px 18px 6px;
  }
  .login-title {
    font-size: 1.5rem;
  }
}
</style>