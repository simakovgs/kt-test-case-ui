<template>
    <div id="auth">
      <form @submit.prevent="onSubmit">
        <label for="username">Имя пользователя:</label>
        <input type="text" id="username" v-model="username" placeholder="Имя пользователя"><br><br>

        <label for="password">Пароль:</label>
        <input type="password" id="password" v-model="password" placeholder="Пароль"><br><br>

        <button type="submit">Авторизоваться</button>
      </form>
      <div>{{ this.result_message }}</div>
    </div>
  </template>
  
  <script>

  export default {
    data() {
      return {
        username: '',
        password: '',
        result_message: null
      };
    },
    methods: {
      async onSubmit() {
        const payload = {
          username: this.username,
          password: this.password
        }

        try {
          const response = await fetch(`${process.env.VUE_APP_API_URL}/user/login`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'          
          },
          body: JSON.stringify(payload)
          })
          
          if (!response.ok) throw new Error('Ошибка отправки данных')
          const data = await response.json();

          localStorage.setItem('accessToken', data.access);
          localStorage.setItem('refreshToken', data.refresh);
          this.result_message = 'Авторизация прошла успешно'
          localStorage.setItem('authStatus', true)

        } catch (error) {
        console.error(error)
        this.result_message = `Ошибка авторизации пользователя ${this.username}`
        localStorage.setItem('accessToken', null);
        localStorage.setItem('refreshToken', null);
        localStorage.setItem('authStatus', false)



      } finally {
        this.isSubmitting = false
      }
      }
    }
  };
  </script>

<style>
#auth {
  max-width: 400px;
  margin: auto;
  padding: 15px;
  background-color: #f8f8ff;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

label {
  display: block;
  font-size: 16px;
  color: #333;
  margin: 5px;
}

input[type="text"],
input[type="password"] {
  width: 90%;
  height: 40px;
  padding: 10px;
  font-size: 16px;
  border: 1px solid #ccc;
  border-radius: 4px;
}



button {
  width: 100%;
  height: 40px;
  background-color: #4CAF50;
  color: white;
  font-size: 16px;
  cursor: pointer;
  border: none;
  border-radius: 4px;
  transition: all 0.3s ease-in-out;
  margin-top: 15px;
}

button:hover {
  background-color: #0056b3;
}

div#result-message {
  text-align: center;
  font-weight: bold;
  color: green;
  margin-top: 10px;
}
</style>
