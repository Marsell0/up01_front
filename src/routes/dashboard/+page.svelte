<script>
  import { browser } from '$app/environment';
  import { writable } from 'svelte/store';
  let userRole = browser ? localStorage.getItem('user_role') : null;
</script>
  
  <h1 class="text-2xl font-bold">Добро пожаловать в панель управления</h1>
  <p>Вы вошли как {userRole}.</p>
  
