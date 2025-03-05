<script>
    import '../../app.css'
    import { onMount } from 'svelte';
    import { goto } from '$app/navigation';
    import { browser } from '$app/environment';
    import { writable } from 'svelte/store';
  
    let email = '';
    let password = '';
    let errorMessage = '';
    let token = writable(browser && localStorage.getItem('token') ? localStorage.getItem('token') : '');
    let userRole = writable(browser && localStorage.getItem('user_role') || '');
  
    async function login() {
      try {
        const response = await fetch('http://localhost:8000/api/token/', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ username: email, password }) 
        });
  
        if (!response.ok) {
          throw new Error('Ошибка авторизации');
        }
  
        const data = await response.json();
  
        if (browser) {
          localStorage.setItem('token', data.access);
          localStorage.setItem('refresh_token', data.refresh);
        }
        
        token.set(data.access);

        const userResponse = await fetch('http://localhost:8000/api/users/me', {
            method: 'GET',
            headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${data.access}`
            }
        });

        if (!userResponse.ok) {
            console.log('Ошибка при загрузке данных пользователя.');
        }

        const userData = await userResponse.json();
        localStorage.setItem('user_role', userData.role);
        userRole.set(userData.role);

        goto('/dashboard');
      } catch (error) {
        errorMessage = 'Ошибка авторизации. Проверьте данные.';
      }
    }
  </script>
  
  
  <div class="flex justify-center items-center h-screen">
    <div class="bg-white p-6 rounded-lg shadow-md w-96">
      <h2 class="text-2xl font-bold mb-4">Вход в систему</h2>
      {#if errorMessage}
        <p class="text-red-500">{errorMessage}</p>
      {/if}
      <input type="email" bind:value={email} placeholder="Email" class="w-full p-2 mb-2 border rounded" />
      <input type="password" bind:value={password} placeholder="Пароль" class="w-full p-2 mb-2 border rounded" />
      <button class="w-full bg-blue-500 text-white py-2 rounded" on:click={login}>Войти</button>
    </div>
  </div>
  