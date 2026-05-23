<template>
  <div class="frontend-layout">
    <header class="navbar-wrapper">
      <div class="navbar-container">
        <div class="brand-section">
          <img class="brand-logo" :src="robotFill" alt="知心心理健康AI助手" />
          <h1 class="brand-name">知心心理健康AI助手</h1>
        </div>

        <input type="checkbox" id="mobile-menu-toggle" class="mobile-menu-checkbox" />
        <label for="mobile-menu-toggle" class="mobile-menu-btn">
          <span class="menu-bar"></span>
          <span class="menu-bar"></span>
          <span class="menu-bar"></span>
        </label>
        <nav class="nav-section">
          <router-link to="/" class="nav-link">首页</router-link>
          <router-link to="/consulation" class="nav-link" v-if="isLogin">AI咨询</router-link>
          <router-link to="/emotional-diary" class="nav-link" v-if="isLogin">情绪日记</router-link>
          <router-link to="/knowledge" class="nav-link">知识库</router-link>
          <el-button size="small" @click="handleLogin">
            {{ isLogin ? '退出登录' : '登录' }}
          </el-button>
          <el-button size="small" @click="handleRegister" v-if="!isLogin">注册</el-button>
        </nav>
      </div>
    </header>

    <main class="main-container">
      <router-view />
    </main>

    <footer class="footer-container">
      <div class="footer-bottom">&copy; 2026 知心心理健康AI助手. All rights reserved.</div>
    </footer>
  </div>
</template>

<script setup>
import { onMounted, ref, watch, nextTick } from 'vue';
import { useRouter, useRoute } from 'vue-router';
import robotFill from '../assets/images/logo.png';

const router = useRouter();
const route = useRoute();
const isLogin = ref(false);

onMounted(() => {
  const token = localStorage.getItem('token');
  isLogin.value = !!token;
});

const closeMobileMenu = () => {
  nextTick(() => {
    const checkbox = document.getElementById('mobile-menu-toggle');
    if (checkbox) checkbox.checked = false;
  });
};

watch(() => route.path, () => {
  closeMobileMenu();
});

const handleLogin = () => {
  closeMobileMenu();
  if (isLogin.value) {
    localStorage.removeItem('token');
    localStorage.removeItem('userInfo');
    isLogin.value = false;
    router.push('/auth/login');
  } else {
    router.push('/auth/login');
  }
};

const handleRegister = () => {
  closeMobileMenu();
  router.push('/auth/register');
};
</script>

<style lang="scss" scoped>
.frontend-layout {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  background-color: #fff;
}

.navbar-wrapper {
  border-bottom: 1px solid #e5e7eb;
}

.navbar-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 10px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.brand-section {
  display: flex;
  align-items: center;
  justify-content: flex-start;
}

.brand-logo {
  width: 50px;
  height: 50px;
  display: block;
}

.brand-name {
  margin: 0 0 0 8px;
  font-size: 24px;
  font-weight: 600;
  color: #333;
}

.nav-section {
  display: flex;
  align-items: center;
  gap: 24px;
}

.nav-link {
  color: #4b5563;
  font-size: 16px;
  font-weight: 500;
  text-decoration: none;
}

.nav-link:hover {
  color: #4a90e2;
}

.main-container {
  flex: 1;
}

.footer-container {
  background: #1f2937;
  color: #fff;
  padding: 15px 0;
}

.footer-bottom {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 10px;
  text-align: center;
}
</style>

<style>
/* ===== 移动端汉堡菜单（纯 CSS 实现，不修改 PC 端代码）===== */
.mobile-menu-checkbox {
  display: none;
}

.mobile-menu-btn {
  display: none;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 40px;
  height: 40px;
  cursor: pointer;
  gap: 5px;
  padding: 8px;
  border-radius: 8px;
  transition: background 0.2s ease;
  flex-shrink: 0;
}

.mobile-menu-btn:hover {
  background: #f0f0f0;
}

.mobile-menu-btn:active {
  background: #e5e7eb;
}

.menu-bar {
  display: block;
  width: 22px;
  height: 2px;
  background: #333;
  border-radius: 2px;
  transition: all 0.3s ease;
}

/* 菜单打开时汉堡图标动画 */
.mobile-menu-checkbox:checked ~ .mobile-menu-btn .menu-bar:nth-child(1) {
  transform: translateY(7px) rotate(45deg);
}

.mobile-menu-checkbox:checked ~ .mobile-menu-btn .menu-bar:nth-child(2) {
  opacity: 0;
}

.mobile-menu-checkbox:checked ~ .mobile-menu-btn .menu-bar:nth-child(3) {
  transform: translateY(-7px) rotate(-45deg);
}

@media (max-width: 768px) {
  .mobile-menu-btn {
    display: flex;
  }

  .nav-section {
    display: none;
  }

  .mobile-menu-checkbox:checked ~ .nav-section {
    display: flex;
    flex-direction: column;
    width: 100%;
    gap: 2px !important;
    padding: 10px 0 6px;
    border-top: 1px solid #e5e7eb;
    animation: slideDown 0.25s ease;
  }

  @keyframes slideDown {
    from { opacity: 0; transform: translateY(-8px); }
    to { opacity: 1; transform: translateY(0); }
  }

  .mobile-menu-checkbox:checked ~ .nav-section .nav-link {
    padding: 10px 14px;
    font-size: 15px;
    border-radius: 8px;
    background: #f8f9fa;
    text-align: center;
    transition: background 0.2s;
  }

  .mobile-menu-checkbox:checked ~ .nav-section .nav-link:active {
    background: #e9ecef;
  }

  .mobile-menu-checkbox:checked ~ .nav-section .el-button {
    width: 100%;
    margin-top: 2px;
    min-height: 40px;
    font-size: 14px;
  }
}
</style>
