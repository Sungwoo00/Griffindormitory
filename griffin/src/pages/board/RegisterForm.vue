<template>
  <div class="register-form">
    <h2>게시물 등록</h2>
    <form @submit.prevent="submitForm">
      <div class="form-group">
        <label for="title">제목</label>
        <input type="text" id="title" v-model.trim="form.title" required />
      </div>
      <div class="form-group">
        <label for="content">내용</label>
        <textarea id="content" v-model.trim="form.content" required></textarea>
      </div>
      <base-btn type="submit">등록</base-btn>
    </form>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';

export default {
  name: 'RegisterForm',
  computed: {
    ...mapGetters('users', ['currentUser']),
    userName() {
      return this.$store.state.users.users[0].name;
    },
    userUid(){
      return this.$store.state.users.users[0].id
    },
    userUniversity() {
      return this.currentUser.university;
    },
  },
  data() {
    return {
      form: {
        title: '',
        content: '',
        author: '',
        university: '',
        userUid:'',
      },
    };
  },
  created() {
    this.form.id = this.userUid;
    this.form.author = this.userName;
    this.form.university = this.userUniversity;
  },
  methods: {
    async submitForm() {
      try {
        await this.$store.dispatch('boards/registerBoard', this.form);
        this.form.title = '';
        this.form.content = '';
        alert('게시물이 성공적으로 등록되었습니다.');
        this.$router.push('/boardlist');
      } catch (error) {
        console.error('게시물 등록 실패:', error);
        alert('게시물 등록에 실패했습니다.');
      }
    },
  },
};
</script>

<style scoped>
.register-form {
  width: 80%;
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 2vh;
  margin-top: 50px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
}

.form-group input,
.form-group textarea,
.form-group select {
  width: 100%;
  padding: 8px;
  box-sizing: border-box;
}
.form-group textarea {
  max-height: 60vh;
  min-height: 30vh;
}

button {
  padding: 10px 20px;
  background-color: #007bff;
  border: none;
  color: white;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}
</style>
