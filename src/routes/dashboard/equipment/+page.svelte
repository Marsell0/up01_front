<script>
  import { onMount } from 'svelte';
  import { browser } from '$app/environment';

  let equipment = [];
  let name = "";
  let status = "";
  let description = "";
  let selectedEquipment = null;
  let userRole = browser ? localStorage.getItem('user_role') : null;
  let token = '';

  onMount(async () => {
    await fetchEquipment();
  });

  async function fetchEquipment() {
    token = browser ? localStorage.getItem('token') : null;
    const response = await fetch('http://localhost:8000/api/equipment/equipment/', {
      headers: {
        'Authorization': `Bearer ${token}`,
        "Content-Type": "application/json"
      }
    });
    equipment = await response.json();
  }

  async function addEquipment() {
    const method = selectedEquipment ? "PUT" : "POST";
    token = browser ? localStorage.getItem('token') : null;
    const url = selectedEquipment 
      ? `http://localhost:8000/api/equipment/equipment/${selectedEquipment.id}/` 
      : "http://localhost:8000/api/equipment/equipment/";

    const response = await fetch(url, {
      method: method,
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${token}`
      },
      body: JSON.stringify({ name, description, status })
    });

    if (response.ok) {
      name = "";
      description = "";
      selectedEquipment = null;
      await fetchEquipment();
    } else {
      alert("Ошибка сохранения!");
    }
  }

  async function reserveEquipment(id) {
    token = browser ? localStorage.getItem('token') : null;
    const response = await fetch(`http://localhost:8000/api/reservation/reservation/`, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${token}`
      },
      body: JSON.stringify({ equipment: id })
    });

    if (response.ok) {
      alert("Оборудование забронировано!");
      await fetchEquipment();
    } else {
      alert("Ошибка при бронировании!");
    }
  }

  function editEquipment(item) {
    selectedEquipment = item;
    name = item.name;
    description = item.description;
    status = item.status;
  }
</script>

<div>
  <h1 class="text-2xl font-bold">Оборудование</h1>

  {#if userRole === "technician"}
    <form on:submit|preventDefault={addEquipment} class="p-4 bg-gray-100 rounded-md my-4">
      <input type="text" bind:value={name} placeholder="Название" class="w-full p-2 border mb-2" required />
      <textarea bind:value={description} placeholder="Описание" class="w-full p-2 border mb-2"></textarea>
      <select bind:value={status} class="w-full p-2 border rounded mt-2">
        <option value="">Выберите статус</option>
        <option value="available">Доступно</option>
        <option value="booked">Забронировано</option>
        <option value="maintenance">На обслуживании</option>
      </select>
      <button type="submit" class="bg-blue-500 text-white p-2 rounded">
        {selectedEquipment ? "Сохранить изменения" : "Добавить оборудование"}
      </button>
    </form>
  {/if}

  <ul class="space-y-2">
    {#each equipment as item}
      <li class="p-4 border rounded-md">
        <h2 class="text-lg font-bold">{item.name}</h2>
        <p>{item.description}</p>
        <p><strong>Статус:</strong> {item.status}</p>

        {#if userRole === "technician"}
          <button on:click={() => editEquipment(item)} class="bg-yellow-500 text-white p-2 rounded mt-2">Редактировать</button>
        {/if}

        {#if userRole === "operator" && item.status === "available"}
          <button on:click={() => reserveEquipment(item.id)} class="bg-green-500 text-white p-2 rounded mt-2">Забронировать</button>
        {/if}
      </li>
    {/each}
  </ul>
</div>
