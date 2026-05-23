<script setup>
// ===== 依赖 =====
import { computed } from "vue";
import { useRouter } from "vue-router";
import { useAdminStore } from "../stores/admin";
import logoUrl from "../assets/images/logo.png";

const router = useRouter();

// ===== 菜单导航 =====
const SelectMenu = (path) => {
  const currentPath = router.options.routes[0].path;
  const fullPath = `${currentPath}/${path}`;
  router.push(fullPath);
};

const isCollapse = computed(() => useAdminStore().iscollapse);
</script>
<template>
  <el-menu
    :collapse="isCollapse"
    :collapse-transition="false"
    default-active="2"
    class="menu-style"
  >
    <!-- Logo区域 -->
    <div class="brand">
      <el-image :src="logoUrl" alt="logo" class="brand-image" />
      <div v-show="!isCollapse" class="info-card">
        <h1 class="brand-title">知心心理健康助手</h1>
        <p class="brand-subtitle">管理后台</p>
      </div>
    </div>
    <el-menu-item
      @click="SelectMenu(item.path)"
      v-for="item in router.options.routes[0].children"
      :key="item.name"
      :index="item.path"
    >
      <el-icon><component :is="item.meta.icon" /></el-icon>
      <span>{{ item.meta.title }}</span>
    </el-menu-item>
  </el-menu>
</template>

<style lang="scss" scoped>
.menu-style {
  height: 100%;
  width: 220px;
  flex-shrink: 0;
  transition: width 0.3s ease;

  &.el-menu--collapse {
    width: 64px;

    .brand {
      padding: 0;
      justify-content: center;
      .brand-image {
        margin-right: 0;
        margin-left: 0;
      }
    }
  }

  .brand {
    display: flex;
    align-items: center;
    padding: 12px 16px;
    margin-bottom: 0;

    .brand-image {
      width: 50px;
      height: 50px;
      margin-right: 12px;
      border-radius: 8px;
      flex-shrink: 0;
    }

    .info-card {
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      flex: 1;

      .brand-title {
        font-size: 14px;
        font-weight: bold;
        margin: 0;
      }

      .brand-subtitle {
        font-size: 11px;
        margin: 0;
      }
    }
  }
}
</style>
