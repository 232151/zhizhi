<template>
  <div class="container">
    <div class="title">
      <div class="back-home" @click="goHome">
        <el-icon><Back /></el-icon>
        <span>返回首页</span>
      </div>
      <div class="title-text">
        <h2>用户注册</h2>
        <p>欢迎加入我们，请填写您的信息</p>
      </div>
    </div>

    <div class="form-container">
      <el-form ref="registerForm" :model="formData" :rules="rules" label-position="top">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="formData.username" placeholder="请输入用户名" />
        </el-form-item>

        <el-form-item label="邮箱" prop="email">
          <el-input v-model="formData.email" placeholder="请输入邮箱" />
        </el-form-item>

        <el-form-item label="昵称" prop="nickname">
          <el-input v-model="formData.nickname" placeholder="请输入昵称（可选）" />
        </el-form-item>

        <el-form-item label="手机号" prop="phone">
          <el-input v-model="formData.phone" placeholder="请输入手机号（可选）" />
        </el-form-item>

        <el-form-item label="密码" prop="password">
          <el-input v-model="formData.password" type="password" show-password placeholder="请输入密码" />
        </el-form-item>

        <el-form-item label="确认密码" prop="confirmPassword">
          <el-input v-model="formData.confirmPassword" type="password" show-password placeholder="请再次输入密码" />
        </el-form-item>

        <el-form-item label="性别">
          <el-select v-model="formData.gender" placeholder="请选择性别">
            <el-option label="男" :value="1" />
            <el-option label="女" :value="2" />
          </el-select>
        </el-form-item>
      </el-form>

      <el-button type="primary" class="btn" @click="handleRegister">注册</el-button>
    </div>
  </div>
</template>

<script setup>
import { reactive, ref, watch } from 'vue';
import { useRouter } from 'vue-router';
import { ElMessage } from 'element-plus';
import { Back } from '@element-plus/icons-vue';
import { register } from '../api/admin';

const router = useRouter();

const goHome = () => {
  router.push('/');
};

const registerForm = ref(null);

const formData = reactive({
  username: '',
  email: '',
  nickname: '',
  phone: '',
  password: '',
  confirmPassword: '',
  gender: null,
  userType: 1
});

const validateConfirmPassword = (rule, value, callback) => {
  if (!value) {
    callback(new Error('请再次输入密码'));
    return;
  }
  if (value !== formData.password) {
    callback(new Error('两次输入的密码不一致'));
    return;
  }
  callback();
};

const rules = reactive({
  username: [
    { required: true, message: '请输入用户名', trigger: 'blur' },
    { min: 3, max: 20, message: '用户名长度必须在 3-20 之间', trigger: 'blur' }
  ],
  email: [
    { required: true, message: '请输入邮箱', trigger: 'blur' },
    { type: 'email', message: '请输入正确的邮箱地址', trigger: ['blur', 'change'] }
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, max: 20, message: '密码长度必须在 6-20 之间', trigger: 'blur' }
  ],
  confirmPassword: [
    { required: true, message: '请再次输入密码', trigger: ['blur', 'change'] },
    { validator: validateConfirmPassword, trigger: ['blur', 'change'] }
  ]
});

// 关键修复：主密码变化后，主动刷新确认密码的校验状态
watch(
  () => formData.password,
  () => {
    if (formData.confirmPassword) {
      registerForm.value?.validateField('confirmPassword');
    }
  }
);

const handleRegister = async () => {
  if (!registerForm.value) return;

  try {
    await registerForm.value.validate();
    const res = await register(formData);
    if (res.code === 200) {
      ElMessage.success('注册成功，请登录');
      router.push('/auth/login');
    } else {
      ElMessage.error(res.message || '注册失败');
    }


  } catch (error) {
    if (error?.message) {
      ElMessage.error(error.message);
    }
  }
};
</script>

<style lang="scss" scoped>
.container {
  width: 100%;
  max-width: 340px;
  padding: 10px;

  .title {
    .title-text {
      text-align: center;
      margin-bottom: 16px;

      h2 {
        font-size: 26px;
        margin-bottom: 6px;
      }

      p {
        font-size: 13px;
        color: #6b7280;
      }
    }
  }

  .form-container {
    margin-top: 8px;

    :deep(.el-form-item) {
      margin-bottom: 12px;
    }

    :deep(.el-input) {
      height: 38px;
    }

    :deep(.el-input__wrapper) {
      padding: 0 12px;
    }

    :deep(.el-form-item__label) {
      padding-bottom: 6px;
      font-size: 13px;
    }

    .btn {
      margin-top: 12px;
      width: 100%;
      height: 40px;
      background: linear-gradient(135deg, #8b7fc0 0%, #b099d4 100%);
      border: none;
      color: #fff;
      font-weight: bold;
      box-shadow: 0 4px 15px rgba(139, 127, 192, 0.4);
    }

    .btn:hover {
      box-shadow: 0 6px 20px rgba(139, 127, 192, 0.6);
      transform: translateY(-2px);
    }

    .btn:active {
      transform: translateY(0);
    }
  }
}
</style>

<style>
/* 返回首页按钮样式（PC + 移动端通用）*/
.back-home {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 15px;
  cursor: pointer;
  color: #6b7280;
  font-size: 14px;
  transition: all 0.3s ease;
}

.back-home:hover {
  color: #374151;
  transform: translateX(-5px);
}

.back-home span {
  margin-left: 6px;
}
</style>
