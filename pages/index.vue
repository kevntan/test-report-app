<template>
  <div class="flex min-h-screen">
    <!-- kiri -->
    <div class="hidden lg:block lg:w-1/2 bg-cover bg-center" style="background-image: url('https://via.placeholder.com/600')">
      <div class="flex items-center justify-center h-full bg-gray-900 bg-opacity-40">
        <div class="text-white text-center">
          <h1 class="text-4xl font-bold">Welcome Back!</h1>
          <p class="mt-2">Sign in to access your account</p>
        </div>
      </div>
    </div>

   <!-- kanan -->
    <div class="flex items-center justify-center w-full p-8 lg:w-1/2 bg-black">
      <div class="w-full max-w-md">
        <img class="mx-auto h-12 w-auto" src="https://tailwindui.com/img/logos/workflow-mark-indigo-600.svg" alt="Workflow" />
        <h2 class="mt-6 text-center text-3xl font-medium text-white">Sign in to your account</h2>

        <div class="mt-8">
          <form @submit.prevent="handleSubmit" class="space-y-6">
            <div>
              <label for="username" class="block text-sm font-light text-white">Username</label>
              <div class="mt-1">
                <input v-model="username" id="username" name="username" placeholder="jon doe" type="text" autocomplete="username" required
                       class="appearance-none block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"/>
              </div>
            </div>
            <div>
              <label for="password" class="block text-sm font-light text-white">Password</label>
              <div class="mt-1">
                <input v-model="password" id="password" name="password" type="password" placeholder="password" autocomplete="current-password" required
                       class="appearance-none block w-full mb-1 px-3 py-2 border border-gray-300 rounded-md shadow-sm placeholder-gray-400 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"/>
              </div>
            </div>
           
            <div>
              <button type="submit"
                      class="group relative w-full flex justify-center py-2 px-4 border border-transparent text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
                Sign in
              </button>
            </div>
            <div v-if="errorMessage" class="text-red-500 text-sm mt-2 text-center">
              {{ errorMessage }}
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const token  = useCookie('token')
const username = ref('')
const password = ref('')
const errorMessage = ref('')


const handleSubmit = async () => {
  try {
    // Replace with your login API endpoint
    const response = await $fetch('https://dummyjson.com/auth/login', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body:{ username: username.value, password: password.value }
    })
    console.log(response)
    token.value = response.token
    navigateTo('/dashboard')
    // if (response.ok) {
    //   Cookies.set('token', data.token, { expires: 2 })
    //   router.push('/about')
    // } else {
    //   errorMessage.value = 'Wrong username or password'
    // }
  } catch (error) {
    console.error('Error logging in:', error)
    errorMessage.value = 'An error occurred. Please try again.'
  }
}
</script>

<style scoped>
</style>
