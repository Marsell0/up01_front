<script>
    import { onMount } from 'svelte';
    import { browser } from '$app/environment';

    let userRole = browser ? localStorage.getItem('user_role') : null;
  
    let users = [];
  
    let newUser = {
      email: '',
      password: '',
      role: ''
    };
    let message = '';
  
    onMount(async () => {
      if (userRole !== 'admin') return;
      
      const token = localStorage.getItem('token');
      const response = await fetch('http://localhost:8000/api/users/users/', {
        headers: { 'Authorization': `Bearer ${token}` }
      });
  
      if (response.ok) {
        users = await response.json();
      }
      else{
        console.log(response)
      }
    });
  
    async function addUser() {
      const token = localStorage.getItem('token');
  
      const response = await fetch('http://localhost:8000/api/users/users/', {
        method: 'POST',
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(newUser)
      });
  
      if (response.ok) {
        message = 'Пользователь успешно добавлен!';
        newUser = { email: '', password: '', role: '' };
  
        // Обновляем список пользователей
        const updatedResponse = await fetch('http://localhost:8000/api/users/users/', {
          headers: { 'Authorization': `Bearer ${token}` }
        });
  
        if (updatedResponse.ok) {
          users = await updatedResponse.json();
        }
      } else {
        message = 'Ошибка при добавлении пользователя';
      }
    }
  </script>
  
  {#if userRole === 'admin'}
    <h1 class="text-2xl font-bold">Управление пользователями</h1>
  
    <div class="mt-4">
      <h2 class="text-lg font-semibold">Добавить нового пользователя</h2>
      <input type="text" bind:value={newUser.username} placeholder="Логин" class="w-full p-2 border rounded mt-2" />
      <input type="password" bind:value={newUser.password} placeholder="Пароль" class="w-full p-2 border rounded mt-2" />
      <select bind:value={newUser.role} class="w-full p-2 border rounded mt-2">
        <option value="">Выберите роль</option>
        <option value="operator">Оператор</option>
        <option value="technician">Техник</option>
        <option value="admin">Администратор</option>
      </select>
      <button on:click={addUser} class="w-full bg-blue-500 text-white py-2 rounded mt-2">Добавить</button>
      {#if message}
        <p class="mt-2 text-green-500">{message}</p>
      {/if}
    </div>
  
    <h2 class="text-lg font-semibold mt-6">Список пользователей</h2>
    {#if users.length > 0}
      <ul>
        {#each users as user}
          <li class="border p-2 mt-2">{user.username} - {user.role}</li>
        {/each}
      </ul>
    {:else}
      <p>Нет пользователей</p>
    {/if}
  {:else}
    <p class="text-red-500">У вас нет доступа к этой странице</p>
  {/if}
  