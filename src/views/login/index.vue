<template>
  <div id="background" class="fixed"></div>
  <div class="bg-backdrop-layout"></div>
  <div class="login-container">
    <div class="login-width md:w-10/12 w-11/12 mx-auto flex justify-between h-full items-center">
      <div class="w-6/12 mx-auto left-panel rounded-l pl-5 pr-5 hidden md:block">
        <div class="logo">
          <img
            src="@/assets/logo.png"
            width=45
          />
          <span>{{"vue-admin-extreme"}}</span></div>
        <div class="slogan flex justify-end">
          <span>---- 开箱即用的高质量中后台管理系统 </span></div>
      </div>
      <div class="md:w-6/12 w-11/12 md:rounded-r mx-auto pl-5 pr-5 pb-10">
        <h2 class="mt-10 text-3xl pb-0 mb-10">登陆</h2>
        <el-form
          ref="loginFormRef"
          :model="loginData"
          :rules="loginRules"
          :hide-lable="true"
        >
          <el-form-item prop="username">
            <el-input
              ref="username"
              v-model="loginData.username"
              class="w-full"
              size="large"
              :placeholder="$t('login.username')"
            >
              <template #prefix>
                <el-icon class="el-input__icon"><user/></el-icon>
              </template>
            </el-input>
          </el-form-item>
          <el-form-item
              prop="password"
              :hide-label="true"
          >
            <el-input
              v-model="loginData.password"
              :placeholder="$t('login.password')"
              :type="passwordVisible === false ? 'password' : 'input'"
              size="large"
              name="password"
              @keyup="checkCapslock"
              @keyup.enter="handleLogin"
              allow-clear
              show-password
            >
              <template #prefix>
                <el-icon class="el-input__icon"><Unlock/></el-icon>
              </template>
            </el-input>
          </el-form-item>

          <el-form-item
            prop="verifyCode"
            :hide-label="true"
          >
            <el-input
              v-model="loginData.verifyCode"
              :placeholder="$t('login.verifyCode')"
              size="large"
              autocomplete="off"
              @keyup.enter="handleLogin"
            >
            <template #append>
            <div class="captcha">
              <img :src="captchaBase64" @click="getCaptcha" />
            </div>
            </template>
            </el-input>
          </el-form-item>

          <el-form-item :hide-label="true" class="mt-5">
            <el-button
              type="primary"
              size="large"
              :loading="loading"
              class="w-full"
              @click.prevent="handleLogin"
            >
              {{ $t('login.login') }}
            </el-button>
          </el-form-item>
          <el-divider orientation="center">{{ $t('其他方式登陆') }}</el-divider>
          <div class="flex w-4/5 pt-2 mx-auto items-stretch justify-around">
            <el-avatar class="other-login github"><svg-icon icon-class="github"/></el-avatar>
            <el-avatar class="other-login wechat"><svg-icon icon-class="wechat"/></el-avatar>
            <el-avatar class="other-login alipay"><svg-icon icon-class="alipay"/></el-avatar>
            <el-avatar class="other-login qq"><svg-icon icon-class="qq"/></el-avatar>
            <el-avatar class="other-login weibo"><svg-icon icon-class="social_sina"/></el-avatar>
          </div>
        </el-form>
      </div>
    </div>
  </div>
</template>


<script setup lang="ts">
import { useI18n } from "vue-i18n";
import router from "@/router";
import LangSelect from "@/components/LangSelect/index.vue";
import SvgIcon from "@/components/SvgIcon/index.vue";

// 状态管理依赖
import { useUserStore } from "@/store/modules/user";
import { useAppStore } from "@/store/modules/app";

// API依赖
import { LocationQuery, LocationQueryValue, useRoute } from "vue-router";
import { getCaptchaApi } from "@/api/auth";
import { LoginData } from "@/api/auth/types";
import {Unlock, User} from "@element-plus/icons-vue";


const appStore = useAppStore();
const userStore = useUserStore();
const route = useRoute();

/**
 * 按钮loading
 */
const loading = ref(false);
/**
 * 是否大写锁定
 */
const isCapslock = ref(false);
/**
 * 密码是否可见
 */
const passwordVisible = ref(false);
/**
 * 验证码图片Base64字符串
 */
const captchaBase64 = ref();

/**
 * 登录表单引用
 */
const loginFormRef = ref(ElForm);

const loginData = ref<LoginData>({
  username: "admin",
  password: "123456",
});

const { t } = useI18n();

const loginRules = computed(() => {
  const prefix = appStore.language === "en" ? "Please enter " : "请输入";
  return {
    username: [
      {
        required: true,
        trigger: "blur",
        message: `${prefix}${t("login.username")}`,
      },
    ],
    password: [
      {
        required: true,
        trigger: "blur",
        validator: passwordValidator,
        message: `${prefix}${t("login.password")}`,
      },
    ],
    verifyCode: [
      {
        required: true,
        trigger: "blur",
        message: `${prefix}${t("login.verifyCode")}`,
      },
    ],
  };
});
// const loginRules = reactive({
//   username: [
//     {
//       required: true,
//       trigger: "blur",
//       message: `请输入${t("login.username")}`,
//     },
//   ],
//   password: [
//     {
//       required: true,
//       trigger: "blur",
//       validator: passwordValidator,
//       message: `请输入${t("login.password")}`,
//     },
//   ],
//   verifyCode: [
//     {
//       required: true,
//       trigger: "blur",
//       message: `请输入${t("login.verifyCode")}`,
//     },
//   ],
// });

/**
 * 密码校验器
 */
function passwordValidator(rule: any, value: any, callback: any) {
  if (value.length < 6) {
    callback(new Error("The password can not be less than 6 digits"));
  } else {
    callback();
  }
}

/**
 * 检查输入大小写状态
 */
function checkCapslock(e: any) {
  const { key } = e;
  isCapslock.value = key && key.length === 1 && key >= "A" && key <= "Z";
}

/**
 * 获取验证码
 */
function getCaptcha() {
  getCaptchaApi().then(({ data }) => {
    const { verifyCodeBase64, verifyCodeKey } = data;
    loginData.value.verifyCodeKey = verifyCodeKey;
    captchaBase64.value = verifyCodeBase64;
  });
}

/**
 * 登录
 */
function handleLogin() {
  loginFormRef.value.validate((valid: boolean) => {
    if (valid) {
      loading.value = true;
      userStore
        .login(loginData.value)
        .then(() => {
          const query: LocationQuery = route.query;

          const redirect = (query.redirect as LocationQueryValue) ?? "/";

          const otherQueryParams = Object.keys(query).reduce(
            (acc: any, cur: string) => {
              if (cur !== "redirect") {
                acc[cur] = query[cur];
              }
              return acc;
            },
            {}
          );

          router.push({ path: redirect, query: otherQueryParams });
        })
        .catch(() => {
          // 验证失败，重新生成验证码
          getCaptcha();
        })
        .finally(() => {
          loading.value = false;
        });
    }
  });
}

onMounted(() => {
  getCaptcha();
});
</script>

<style scoped lang="less">
#background {
  top: 0; left: 0;
  z-index: 1;
  width: 100%;
  height: 100%;
}
.bg-backdrop-layout {
  top: 0; left: 0;
  position: fixed;
  width: 100%; height: 100%;
  z-index: 2;
  backdrop-filter: blur(25px);
}
.login-container {
  width: 100%;
  height: 100%;
  position: absolute;
  background-size: cover;
  z-index: 3;
  .login-width {
    max-width: 950px;
    background: #fff;
    padding: 10px;
    height: 500px;
    position: relative;
    top: 50%;
    margin-top: -255px;
    border-radius: 10px;
  }

  .left-panel {
    height: 491px;
    background-image: url(@/assets/login_picture.svg);
    background-repeat: no-repeat;
    background-position: center 60px;
    background-size:contain;
  }

  .logo {
    display: flex; margin-top: 20px; color: #333;
    span {
      font-size: 28px; margin-left: 15px;
      color: rgb(var(--primary-6));
    }
  }
  .slogan {
    font-size: 16px; line-height: 50px;
  }

  :deep(.arco-input-append) {
    padding: 0 !important;
  }

  .other-login{
    cursor: pointer;
  }

  .qq:hover, .alipay:hover {
    background: #165DFF;
  }
  .wechat:hover {
    background: #0f9c02;
  }

  .weibo:hover {
    background: #f3ce2b;
  }

  .github:hover {
    background: #000000 ;
  }
}
</style>
