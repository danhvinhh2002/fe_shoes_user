<template>
  <div class="flex flex-col py-[50px] items-center justify-center bg-cover" style="
        background-image: url('https://goodfit.vn/wp-content/uploads/2021/01/gang-tay-the-thao-chong-nang-6.jpg');
      ">
    <div
      class="flex flex-col bg-white opacity-90 shadow-md px-4 sm:px-6 md:px-8 lg:px-10 py-8 rounded-md w-full max-w-md">
      <h1 class="text-center text-2xl font-bold uppercase">
        Đăng ký tài khoản mới
      </h1>
      <div class="mt-3">
        <a-form ref="formRef" class="w-full sm:max-w-[300px] lg:max-w-[500px] relative" style="margin: 0 auto"
          :model="formState" name="basic" layout="vertical">
          <div class="flex flex-col">
            <a-form-item label="Tài khoản" name="userName"
              :rules="[{ required: true, message: 'Tài khoản không được để trống!' }]">
              <a-input class="h-10 text-base"
                @change="formState.userName = formState.userName.replace(/[^a-zA-Z0-9@]/g, '').toLowerCase()"
                v-model:value.trim="formState.userName" placeholder="Tài khoản ..." />
            </a-form-item>
          </div>
          <div class="flex flex-col">
            <a-form-item label="Email" name="email" :rules="[{ validator: handleValidateTypeEmail }]">
              <a-input class="h-10 text-base" v-model:value.trim="formState.email"
                @change="formState.email = formState.email.replace(/^\s+/, '')" placeholder="Email ..." />
            </a-form-item>
          </div>
          <div class="flex flex-col">
            <a-form-item label="Số điện thoại" name="phone"
              :rules="[{ validator: handleValidateTypePhone }, { required: true, message: 'Số điện thoại không được để trống!' }]">
              <a-input class="h-10 text-base" v-model:value="formState.phone" :maxlength="10"
                @change="formState.phone = formState.phone.replace(/[^0-9@]/g, '')" placeholder="Số điện thoại ..." />
            </a-form-item>
          </div>
          <div class="flex flex-col">
            <a-form-item label="Họ và tên" name="name"
              :rules="[{ required: true, message: 'Họ và tên không được để trống!' }]">
              <a-input class="h-10 text-base" v-model:value="formState.name"
                @change="formState.name = _removeSpecialChars(formState.name).replace(/^\s+/, '')" placeholder="Họ và tên ..." />
            </a-form-item>
          </div>
          <div class="flex flex-col mb-3">
            <a-form-item style="margin-bottom: 0" label="Mật khẩu" name="password"
              :rules="[{ required: true, message: 'Mật khẩu không được để trống!' }, { min: 6, message: 'Mật khẩu cần phải tối thiểu 6 ký tự!' }]">
              <a-input-password class="h-10 text-base" v-model:value="formState.password" @change="formState.password = formState.password.toLowerCase()" placeholder="Mật khẩu ..." />
            </a-form-item>

          </div>

          <div class="flex w-full">
            <a-button html-type="button" @click="handleSubmit"
              class="flex z-auto items-center justify-center focus:outline-none h-10 hover:text-white text-white text-sm sm:text-base bg-blue-600 hover:bg-blue-700 rounded py-2 w-full transition duration-150 ease-in">
              <span class="justify-center uppercase">Đăng ký tài khoản mới</span>
            </a-button>
          </div>
        </a-form>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useCounterStore } from '@/stores/counter'
import { authService } from "../services/AuthService";
import { message } from "ant-design-vue";

const _removeSpecialChars = inject("removeSpecialChars", (val: string) => val);
const _router = useRouter();
const storeCounter = useCounterStore()

const increase = () => storeCounter.increment();
// ✅ this one will be reactive
// 💡 but you could also just use `store.doubleCount` directly
const doubleValue = computed(() => storeCounter.doubleCount)



onMounted(() => {

  console.log("signup page mounted: ", storeCounter);
});
onsubmit = (e) => {
  e.preventDefault();
  console.log("submit");
  increase();
  console.log("signup page mounted: ", storeCounter);
}
const formState = reactive({
  role: "ROLE_USER",
  userName: '',
  password: '',
  email: '',
  phone: '',
  name: '',
});
const formRef = ref(null);

const handleSubmit = () => {
  formRef.value.validate().then(() => {
    // Check for empty username and password
    if (formState.userName.trim() === '' || formState.password.trim() === '') {
      notification.error({
        message: 'Vui lòng nhập tài khoản và mật khẩu.'
      });
      return;
    }
    console.log("formState: ", formState);
    authService.register(formState)
      .then((res) => {
        console.log("res: ", res);
        if (res.status == 200) {
          notification.success({
            message: "Đăng ký thành công"
          });
          _router.push("/dang-nhap");
        } else {
          notification.error({
            message: "Đăng ký thất bại"
          });
        }
      })
      .catch((err) => {
        if (err?.response?.data?.code == 21)
          notification.error({
            message: "Tài khoản này đã được đăng ký."
          });
        else if (err?.response?.data?.code == 22)
          notification.error({
            message: "Email này đã được đăng ký."
          });
        else if (err?.response?.data?.code == 23)
          notification.error({
            message: "Số điện thoại này đã được đăng ký."
          });
        else {
          notification.error({
            message: "Đăng ký thất bại!"
          });
        }
      });
  });
}

const handleValidateTypePhone = (rule, value) => {
  if (value && !/((09|03|07|08|05)+([0-9]{8})\b)/g.test(value)) {
    return Promise.reject('Số điện thoại không đúng!');
  }
  return Promise.resolve();
};
const handleValidateTypeEmail = (rule, value) => {
  if (value && !/^[a-zA-Z0-9_.+-]+@[a-zA-Z0-9-]+\.[a-zA-Z0-9-.]+$/g.test(value)) {
    return Promise.reject('Email không hợp lệ!');
  }
  return Promise.resolve();
};

</script>