
<template>
  <div class="login-container">
    <div class="logo-container">
      <img src="../../assets/images/Logo.png" class="logo" />

      <h2>GRIFFIN</h2>
    </div>
    <div class="user">
      <form @submit.prevent="handleSubmit">
        <div class="user-box">
          <input type="email" v-model="email" required />
          <label>이메일</label>
        </div>
        <div class="user-box">
          <input type="password" v-model="password" required />
          <label>비밀번호</label>
        </div>
        <div>
          <base-btn type="submit" class="submit-btn">
            Login
          </base-btn>
        </div>
        
        <div class='signup'>
          <router-link to='/signup'>회원가입</router-link>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
import { auth, db } from "@/firebase/config";
import {
  signInWithEmailAndPassword,
  createUserWithEmailAndPassword,
} from "firebase/auth";
import { ref, set } from "firebase/database";
// import BaseBtn from "../UI/BaseBtn.vue";

export default {
  // components: { BaseBtn },
  name: "LoginForm",
  data() {
    return {
      isLogin: true,
      name: "",
      university: "",
      studentId: "",
      gender: "",
      email: "",
      password: "",
      confirmPassword: "",
      userId: null,
    };
  },
  methods: {
    validateForm() {
      if (!this.email) {
        alert("이메일을 입력하세요.");
        return false;
      }
      if (!/\S+@\S+\.\S+/.test(this.email)) {
        alert("유효한 이메일 주소를 입력하세요.");
        return false;
      }
      if (!this.password) {
        alert("비밀번호를 입력하세요.");
        return false;
      }
      if (!this.isLogin) {
        if (!this.name) {
          alert("이름을 입력하세요.");
          return false;
        }
        if (!this.university) {
          alert("대학교를 입력하세요.");
          return false;
        }
        if (!this.studentId) {
          alert("학번 또는 수험 응시 번호를 입력하세요.");
          return false;
        }
        if (!this.gender) {
          alert("성별을 선택하세요.");
          return false;
        }
        if (this.password !== this.confirmPassword) {
          alert("비밀번호와 비밀번호 확인이 일치하지 않습니다.");
          return false;
        }
      }
      return true;
    },
    async handleSubmit() {
      if (!this.validateForm()) return;
      if (this.isLogin) {
        try {
          const userCredential = await signInWithEmailAndPassword(
            auth,
            this.email,
            this.password
          );
          this.$store.commit("users/setUserId", userCredential.user.uid);
          await this.$store.dispatch("users/fetchUserInitialData", {
            uid: userCredential.user.uid,
          });

          this.$emit("login-success", userCredential.user);
          this.$router.push("/info");
        } catch (error) {
          console.error("로그인 오류: ", error);
          this.handleAuthError(error);
        }
      } else {
        try {
          const userCredential = await createUserWithEmailAndPassword(
            auth,
            this.email,
            this.password
          );
          const user = userCredential.user;
          const userRef = ref(db, "users/" + user.uid);
          await set(userRef, {
            name: this.name,
            university: this.university,
            studentId: this.studentId,
            gender: this.gender,
            email: this.email,
            createdAt: new Date().toISOString(),
          });
          this.$store.commit("users/setUserId", user.uid);
          await this.$store.dispatch("users/fetchUserInitialData", {
            uid: user.uid,
          });

          alert("회원가입 성공 하였습니다!");
          this.$emit("login-success", user);
          this.$router.push("/info");
        } catch (error) {
          console.error("회원가입 오류: ", error);
          this.handleAuthError(error);
        }
      }
    },
    handleAuthError(error) {
      switch (error.code) {
        case "auth/email-already-in-use":
          alert("이미 사용 중인 이메일입니다.");
          break;
        case "auth/invalid-email":
          alert("유효하지 않은 이메일 주소입니다.");
          break;
        case "auth/operation-not-allowed":
          alert("이메일/비밀번호 인증이 현재 사용 설정되어 있지 않습니다.");
          break;
        case "auth/weak-password":
          alert("비밀번호가 너무 약합니다. 더 복잡한 비밀번호를 사용하세요.");
          break;
        case "auth/user-disabled":
          alert("이 계정은 비활성화되었습니다.");
          break;
        case "auth/user-not-found":
          alert("사용자를 찾을 수 없습니다. 이메일을 확인하세요.");
          break;
        case "auth/wrong-password":
          alert("잘못된 비밀번호입니다. 비밀번호를 다시 확인하세요.");
          break;
        case "auth/network-request-failed":
          alert("네트워크 요청에 실패했습니다. 인터넷 연결을 확인하세요.");
          break;
        default:
          alert("알 수 없는 오류가 발생했습니다. 다시 시도해 주세요.");
      }
    },
    logIn() {
      this.email = "audwognl@gmail.com";
      this.password = "123123";
      this.handleSubmit();
    },
  },
};
</script>

<style scoped>
html,
body {
  height: 100%;
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #f2f2f2;
}

.logo-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 50px;
}

.logo {
  width: 80px;
  height: 80px;
  margin-bottom: 10px;
  filter: invert(100%);
}

/* .login-box {
  width: 400px;
  padding: 40px;
  background: rgba(0, 0, 0, 0.5);
  box-sizing: border-box;
  box-shadow: 0 15px 25px rgba(0, 0, 0, 0.6);
  border-radius: 10px;
} */
.signup{
  margin-top: 20px;
  text-align: center;
  font-size: 14px;
  text-decoration: underline;

}
h2 {
  margin: 0 0 30px;
  padding: 0;
  color: #fff;
  text-align: center;
}

.user{
  padding: 20px;
}

.user-box {
  position: relative;
}

.logo-container {
  height: 40vh;
  background: linear-gradient(130deg, #795ade, #6471e5);
  justify-content: center;
}

.user-box input {
  width: 100%;
  padding: 10px 0;
  font-size: 16px;
  /* color: #fff; */
  margin-bottom: 30px;
  border: none;
  border-bottom: 1px solid #403e3e;
  outline: none;
  background: transparent;
}

.user-box label {
  position: absolute;
  top: 0;
  left: 0;
  padding: 10px 0;
  font-size: 16px;
  /* color: #fff; */
  pointer-events: none;
  transition: 0.5s;
}
.login-box .user-box select {
  width: 100%;
  padding: 10px 0;
  font-size: 16px;
  color: #fff;
  margin-bottom: 30px;
  border: none;
  border-bottom: 1px solid #fff;
  outline: none;
  background: transparent;
  appearance: none;
}

.user-box input:focus ~ label,
.user-box input:valid ~ label,
.user-box select:focus ~ label,
.user-box select:valid ~ label {
  top: -20px;
  left: 0;
  color: #87acf6;
  font-size: 12px;
}

.login-box form a {
  position: relative;
  display: inline-block;
  padding: 10px 20px;
  color: #87acf6;
  font-size: 16px;
  text-decoration: none;
  text-transform: uppercase;
  overflow: hidden;
  transition: 0.5s;
  margin-top: 40px;
  letter-spacing: 4px;
  cursor: pointer;
}
</style>

