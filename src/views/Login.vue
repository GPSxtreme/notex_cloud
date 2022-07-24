<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import { useSignInEmailPassword, useSignUpEmailPassword } from "@nhost/vue";

const router = useRouter();
const isRegister = ref(false);

const email = ref("");
const password = ref("");

const { signInEmailPassword } = useSignInEmailPassword();
const { signUpEmailPassword } = useSignUpEmailPassword();

const registerOrLogin = async () => {
  if (!email.value || !password.value) {
    return alert("Please fill in all fields");
  }

  const res = isRegister.value
    ? await signUpEmailPassword(email.value, password.value)
    : await signInEmailPassword(email.value, password.value);
  console.log(res);
  if (res.isError) {
    return alert(res.error.message);
  }
  if (res.isSuccess) router.push("/");
};
</script>

<template>
  <main>
    <h1 class="logo"><span class="text-green-500">U-</span>Notes📃</h1>
    <form @submit.prevent="registerOrLogin">
      <div class="shadow-xl p-6 bg-green-400 login-box">
        <h3
          class="underline underline-offset-4 decoration-black decoration-dashed text-2xl font-bold uppercase mb-4 text-center"
        >
          Login or Register
        </h3>
        <label class="block mb-4"
          ><input
            type="checkbox"
            v-model="isRegister"
            class="mr-2"
          />Register?</label
        >
        <label class="block mb-4 mt-4"
          ><span class="block text-sm uppercase mb-2">Email</span></label
        >
        <input
          type="email"
          v-model="email"
          placeholder="Enter your email"
          class="block w-full p-2 border border-gray-300 rounded-md text-slate-800"
        />
        <label class="block mb-4 mt-4"
          ><span class="block text-sm uppercase mb-2">Password</span></label
        >
        <input
          type="password"
          v-model="password"
          placeholder="Enter your password"
          class="block w-full p-2 border border-gray-300 rounded-md text-slate-800"
        />
        <input
          type="submit"
          value="Login or Register"
          class="text-green-800 hover:underline cursor-pointer mt-4"
        />
      </div>
    </form>
  </main>
</template>
<style>
.login-box {
  box-shadow: 2.8px 2.8px 2.2px rgba(0, 0, 0, 0.02),
    6.7px 6.7px 5.3px rgba(0, 0, 0, 0.028),
    12.5px 12.5px 10px rgba(0, 0, 0, 0.035),
    22.3px 22.3px 17.9px rgba(0, 0, 0, 0.042),
    41.8px 41.8px 33.4px rgba(0, 0, 0, 0.05),
    100px 100px 80px rgba(0, 0, 0, 0.07);
}
.logo {
  margin: 10px auto;
  margin-bottom: 40px;
  text-align: center;
  font-size: 2.5rem;
  background-color: rgba(128, 128, 128, 0.558);
  padding: 10px 20px;
}
</style>
