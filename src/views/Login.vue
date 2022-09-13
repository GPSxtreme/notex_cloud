<script setup>
//notex cloud☁☁
import { ref } from "vue";
import { useRouter } from "vue-router";
import { useSignInEmailPassword, useSignUpEmailPassword } from "@nhost/vue";

const router = useRouter();
const isRegister = ref(false);

let email = ref("");
let password = ref("");
let confirmPassword = ref("");
let doMatch = ref(false);
let emailSent = ref(false);

const { signInEmailPassword } = useSignInEmailPassword();
const { signUpEmailPassword } = useSignUpEmailPassword();

const match = () => {
  if (password.value !== confirmPassword.value) {
    doMatch.value = false;
    return false;
  } else {
    doMatch.value = true;
    return true;
  }
};

const emailSend = () => {
  emailSent.value = true;
};

const registerOrLogin = async () => {
  if (!email.value || !password.value) {
    s;
    return alert("Please fill in all fields");
  }
  if (match() && isRegister.value == true) {
    emailSent.value = false;
    var res = await signUpEmailPassword(email.value, password.value);
    const { sendEmail, isLoading, isSent, isError, error } =
      useSendVerificationEmail();
    await sendEmail({
      email: email.value,
    });
  } else if (isRegister.value == false) {
    var res = await signInEmailPassword(email.value, password.value);
  }
  if (res.isSuccess) router.push("/");
};
</script>

<template>
  <main>
    <h1 class="logo"><span class="text-green-500">U-</span>Notes📃</h1>
    <form @submit.prevent="registerOrLogin">
      <div class="login-box">
        <h3
          class="underline underline-offset-4 decoration-black decoration-dashed text-2xl font-bold uppercase mb-4 text-center"
        >
          Login or Register
        </h3>
        <label class="block mb-4"
          ><input type="checkbox" v-model="isRegister" class="mr-2" />New
          user?</label
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
        <label class="block mb-4 mt-4" v-if="isRegister"
          ><span class="block text-sm uppercase mb-2"
            >confirm password</span
          ></label
        >
        <input
          type="password"
          v-model="confirmPassword"
          v-if="isRegister"
          placeholder="Enter your password"
          class="block w-full p-2 border border-gray-300 rounded-md text-slate-800"
        />
        <label class="block mb-4 mt-4" v-if="doMatch === false && isRegister"
          ><span class="block text-sm uppercase mb-2 text-red-500 font-bold">
            passwords don't match!!</span
          ></label
        >
        <input
          type="submit"
          @click="emailSend"
          value="Login or Register"
          class="text-green-800 hover:underline cursor-pointer mt-4"
        />
        <label class="block mb-4 mt-4" v-if="emailSent && isRegister"
          ><span class="block text-sm uppercase mb-2 text-indigo-500"
            >Please confirm your email by clicking the link sent to your
            inbox!🫡</span
          ></label
        >
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
  width: 80%;
  margin: 0 auto;
  background-color: rgb(66, 235, 74);
  padding: 20px;
}
.logo {
  margin: 10px auto;
  margin-bottom: 40px;
  text-align: center;
  font-size: 2.5rem;
  background-color: rgba(128, 128, 128, 0.558);
  padding: 10px 20px;
  width: 80%;
  padding: 7px 10px;
  border: 2px dashed rgb(35, 37, 85);
  box-shadow: 6.7px 6.7px 5.3px rgba(0, 0, 0, 0.041),
    22.3px 22.3px 17.9px rgba(0, 0, 0, 0.059),
    100px 100px 80px rgba(0, 0, 0, 0.12);
}
</style>
