<template>
  <div v-if="isAddFormShowed">
    <CreateUser @addUser="UpdateUsers"/>
  </div>
  <div v-if="isEditFormShowed">
    <EditUser @editUser="UpdateUsers" v-bind:user="userToEdit" />
  </div>
  <div>
    <table class="table">
      <thead>
        <tr>
          <th>ID</th>
          <th>Role</th>
          <th>Name</th>
          <th>Email</th>
          <th>Email Verified</th>
          <th>Created At</th>
          <th>Actions <button @click="addUser()" :disabled="isEditFormShowed">Add User</button></th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in paginatedUsers" :key="user.id">
          <td>{{ user.id }}</td>
          <td>{{ user.role }}</td>
          <td>{{ user.name }}</td>
          <td>{{ user.email }}</td>
          <td>{{ user.email_verified_at ? 'Yes' : 'No' }}</td>
          <td>{{ user.created_at }}</td>
          <td>
            <button @click="editUser(user)" :disabled="isAddFormShowed">Edit</button>
            <button @click="deleteUser(user)">Delete</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Pagination controls -->
    <div class="pagination">
      <button @click="prevPage" :disabled="currentPage === 1">Previous</button>
      <span>Page {{ currentPage }} of {{ totalPages }}</span>
      <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
    </div>

  </div>
</template>

<script setup>
  import axios from 'axios';
  import { ref, onMounted, computed } from 'vue';
  import { useAuthStore } from '../../stores/auth';
  import CreateUser from './CreateUser.vue'
  import EditUser from './EditUser.vue'
  
  let isAddFormShowed = ref(false)
  let isEditFormShowed = ref(false)

  const authStore = useAuthStore()

  const users = ref([])
  let userToEdit = ref(null)
  const perPage = ref(10); 
  let currentPage = ref(1);

  onMounted(async () => {
    await authStore.getUser()
    const response = await axios.get('/api/users');
    users.value = response.data;
  })

    const paginatedUsers = computed(() => {
      const startIndex = (currentPage.value - 1) * perPage.value;
      const endIndex = startIndex + perPage.value;
      return users.value.slice(startIndex, endIndex);
    });

  const totalPages = computed(() => Math.ceil(users.value.length / perPage.value));

  const prevPage = () => {
    if (currentPage.value > 1) {
      currentPage.value -= 1;
    }
  };

  const nextPage = () => {
    if (currentPage.value < totalPages.value) {
      currentPage.value += 1;
    }
  };

  const addUser = () => {
    isAddFormShowed.value = true
    
  }

  const UpdateUsers = (userData) => {
    if(isAddFormShowed.value){
      isAddFormShowed.value = false
      users.value.push(userData);
    }
    if(isEditFormShowed.value){
      isEditFormShowed.value = false

      users.value[userToEdit.id-1].name = userData.name
      users.value[userToEdit.id-1].role = userData.role
      users.value[userToEdit.id-1].email = userData.email
      
    }
  }



  const editUser = (user) => {
    userToEdit = user
    isEditFormShowed.value = true
  }
  const deleteUser = async (user) => {
    await axios.delete(`/api/users/${user.id}`);
    users.value.splice(users.value.indexOf(user), 1);

  }


</script>
 
<style scoped>
  .table {
    width: 100%;
    border-collapse: collapse;
    margin: 20px 0;
    font-size: 16px;
  }

  .table th {
    background-color: #451952;
    text-align: left;
    padding: 12px;
    color: #fff;
  }

  .table td {
    border: 1px solid #ddd;
    padding: 12px;
  }

  .table tr:nth-child(even) {
    background-color: #662549;
  }

  .table button {
    padding: 6px 12px;
    margin-right: 5px;
    cursor: pointer;
    background-color: #AE445A;
    color: #fff;
    border: none;
    border-radius: 4px;
  }

  .table button:hover {
    background-color: #F39F5A;
  }
</style>