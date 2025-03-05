<script>
	import '../../app.css'
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';
  
	let token = null;
	let userRole = null;
  
	onMount(() => {
	  if (typeof window !== 'undefined') {
		token = localStorage.getItem('token');
		userRole = localStorage.getItem('user_role');
  
		if (!token) {
		  goto('/login');
		}
	  }
	});
  
	function logout() {
	  localStorage.removeItem('token');
	  localStorage.removeItem('user_role');
	  goto('/login');
	}
  </script>
  
  <nav class="bg-gray-800 p-4 text-white flex justify-between">
	<div class="flex space-x-4">
	  <a href="/dashboard" class="hover:underline">Главная</a>
	  <a href="/dashboard/equipment" class="hover:underline">Оборудование</a>
	  {#if userRole === 'admin'}
		<a href="/dashboard/users" class="hover:underline">Пользователи</a>
	  {/if}
	</div>
	<button on:click={logout} class="bg-red-500 px-4 py-2 rounded">Выйти</button>
  </nav>
  
  <main class="p-4">
	<slot />
  </main>
  