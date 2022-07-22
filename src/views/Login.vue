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
    <h1 class="text-center text-3xl font-bold mb-8">Notes with Auth👌</h1>
    <form @submit.prevent="registerOrLogin">
      <h3 class="text-lg font-bold uppercase mb-4">Login or Register🖋️</h3>
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
        class="text-green-500 hover:underline cursor-pointer mt-4"
      />
    </form>
  </main>
</template>
