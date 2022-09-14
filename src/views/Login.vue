<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";
import { useSignInEmailPassword, useSignUpEmailPassword } from "@nhost/vue";

const router = useRouter();
const isRegister = ref(false);

let email = ref("");
let password = ref("");
let confirmPassword = ref("");
let doMatch = ref(true);
let emailSent = ref(false);
let emailErr = ref(false);
let emailExists = ref(false);

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
  if (doMatch.value && isRegister && !emailErr && !emailExists)
    emailSent.value = true;
};
const registerOrLogin = async () => {
  if (!email.value || !password.value) {
    return alert("Please fill in all fields");
  } else if (email.value && password.value) {
    if (isRegister.value) {
      match();
    }
    if (doMatch.value && isRegister.value) {
      var res = await signUpEmailPassword(email.value, password.value);
      if (res.isError) {
        emailExists.value = true;
      }
      if (!emailExists.value) {
        emailSent.value = true;
      }
      if (res.isSuccess) {
        const { sendEmail, isLoading, isSent, isError, error } =
          useSendVerificationEmail();
        await sendEmail({
          email: email.value,
        });
      }
    } else if (isRegister.value == false) {
      var res = await signInEmailPassword(email.value, password.value);
      if (res.isError) {
        emailErr.value = true;
      }
    }
    if (res.isSuccess) {
      router.push("/");
    }
  }
};
</script>

<template>
  <main>
    <h1 class="logo"><span class="logo1">Notex </span>cloud☁</h1>
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
          class="inputText block w-full text-slate-800"
        />
        <label class="block mb-4 mt-4"
          ><span class="block text-sm uppercase mb-2">Password</span></label
        >
        <input
          type="password"
          v-model="password"
          placeholder="Enter your password"
          class="inputText block w-full text-slate-800"
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
          class="inputText block w-full text-slate-800"
        />
        <label class="block mb-4 mt-4" v-if="!doMatch && isRegister"
          ><span class="block text-sm uppercase mb-2 text-red-500 font-bold">
            passwords don't match!!</span
          ></label
        >
        <label class="block mb-4 mt-4" v-if="emailErr && !emailExists"
          ><span class="block text-sm uppercase mb-2 text-red-500 font-bold">
            Incorrect email or password!!</span
          ></label
        >
        <label class="block mb-4 mt-4" v-if="emailExists"
          ><span class="block text-sm uppercase mb-2 text-red-500 font-bold">
            Email is already registered!</span
          ></label
        >
        <input
          type="submit"
          @click="emailSend"
          value="Login or Register"
          class="text-green-800 hover:underline cursor-pointer mt-4"
        />
        <label class="block mb-4 mt-4" v-if="emailSent && isRegister"
          ><span class="block text-xl uppercase mb-2 text-indigo-500"
            >Email verification sent to your inbox!📧</span
          ></label
        >
      </div>
    </form>
    <div class="footer">
      <h1 class="footer-txt">
        ✏️ Designed and 🛠️Built by
        <a
          href="https://prudhvisuraaj.netlify.app/"
          class="footer-redirect"
          target="blank"
        >
          Prudhvi</a
        >
      </h1>
    </div>
  </main>
</template>
<style>
@import url("https://fonts.googleapis.com/css2?family=Inter&family=Pacifico&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Bebas+Neue&display=swap");
@import url("https://fonts.googleapis.com/css2?family=Inconsolata:wght@500&display=swap");

main {
  background-color: #c8eed9;
  min-height: 100vh;
}

.login-box {
  box-shadow: 2.8px 2.8px 2.2px rgba(0, 0, 0, 0.02),
    6.7px 6.7px 5.3px rgba(0, 0, 0, 0.028),
    12.5px 12.5px 10px rgba(0, 0, 0, 0.035),
    22.3px 22.3px 17.9px rgba(0, 0, 0, 0.042),
    41.8px 41.8px 33.4px rgba(0, 0, 0, 0.05),
    100px 100px 80px rgba(0, 0, 0, 0.07);
  width: 80%;
  margin: 0 auto;
  background-color: #fff1bd;
  font-family: "Inconsolata", monospace;
  font-size: 1.2rem;
  color: #146356;

  padding: 20px;
}
.logo {
  margin: 10px auto;
  margin-bottom: 40px;
  text-align: center;
  font-family: "Bebas Neue", cursive;
  font-size: 3rem;
  letter-spacing: 3px;
  background-color: #fff1bd;
  width: 80%;
  color: #2eb2ff;
  padding: 7px 10px;
  border: 2px dashed rgb(35, 37, 85);
  box-shadow: 6.7px 6.7px 5.3px rgba(0, 0, 0, 0.041),
    22.3px 22.3px 17.9px rgba(0, 0, 0, 0.059),
    100px 100px 80px rgba(0, 0, 0, 0.12);
}
.logo1 {
  font-family: "Pacifico", cursive;
  letter-spacing: none;
  color: #146356;
}
.inputText {
  font-family: inherit;
  border: 0;
  border-bottom: 2px solid black;
  outline: 0;
  color: #146356;
  padding: 7px 7px;
  background: transparent;
  transition: border-color 0.2s;
}
.inputText:focus {
  background: #f3c79295;
  transition: 0.2s;
  color: black;
  font-size: 1.1rem;
}
.footer {
  background-color: #fff1bd;
  padding: 10px;
  text-align: center;
  line-height: 1.5;
  position: fixed;
  left: 0;
  bottom: 0;
  width: 100%;
  border-top: 2px dashed rgb(35, 37, 85);
}
.footer-txt {
  color: #146356;
  font-size: 1.2rem;
  padding: 10px;
  font-family: "Pacifico", cursive;
}
.footer-redirect {
  color: #2eb2ff;
  text-decoration: underline;
  font-family: "Bebas Neue", cursive;
  font-size: 1.5rem;
}
.footer-redirect:hover {
  color: rgb(72, 0, 255);
}
@media only screen and (max-width: 470px) {
  .logo {
    font-size: 150%;
  }
  .footer-txt,
  .footer-redirect {
    font-size: 100%;
  }
}
</style>
