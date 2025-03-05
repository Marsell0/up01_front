<script>
    import { onMount } from "svelte";
    import { goto } from "$app/navigation";
    import { writable } from 'svelte/store';
    import { browser } from '$app/environment';
  
    let equipmentList = [];
    let errorMessage = "";
    let userRole = writable(browser && localStorage.getItem('user_role') ? localStorage.getItem('user_role') : '');
  
    async function fetchEquipment() {
      try {
        const token = writable(browser && localStorage.getItem('token') ? localStorage.getItem('token') : '');
        const response = await fetch("http://localhost:8000/api/equipment/", {
            method: "GET",
            headers: {
            "Authorization": `Bearer ${token}`,
            "Content-Type": "application/json"
            }
        });
        equipmentList = response.data;
      } catch (error) {
        errorMessage = "Ошибка загрузки данных.";
      }
    }
  
    onMount(() => {
      if (userRole !== "operator" && userRole !== "technician") {
        // goto("/login"); // Перенаправление если нет доступа
        console.log(userRole)
      } else {
        fetchEquipment();
      }
    });
  </script>
  
  <div class="p-6">
    <h2 class="text-2xl font-bold mb-4">Список оборудования</h2>
  
    {#if errorMessage}
      <p class="text-red-500">{errorMessage}</p>
    {/if}
  
    <table class="w-full border-collapse border border-gray-300">
      <thead>
        <tr class="bg-gray-200">
          <th class="border p-2">Название</th>
          <th class="border p-2">Статус</th>
          <th class="border p-2">Локация</th>
        </tr>
      </thead>
      <tbody>
        {#each equipmentList as equipment}
          <tr class="border">
            <td class="border p-2">{equipment.name}</td>
            <td class="border p-2">{equipment.status}</td>
            <td class="border p-2">{equipment.location}</td>
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
  