<script>
    import {appwriteUser} from '../lib/backend';
    
    import { ID } from 'appwrite';
  
    export let email = '';
    let password = '';
    let errorMessage = '';
  
    $: disabled = email === '' || password === '';
  
  function login() {
    disabled = true
    const promise = appwriteUser.createEmailSession(email, password);
    promise.then((response) => {
      console.log(response);
      // set the response to a cookie that expires after 30 days
      document.cookie = `user=${response.userId}; expires=${new Date(Date.now() + 30 * 24 * 60 * 60 * 1000).toUTCString()}; path=/`;
      // the user name is the text before the @ symbol in the email address
      document.cookie = `userName=${response.providerUid.split('@')[0]}; expires=${new Date(Date.now() + 30 * 24 * 60 * 60 * 1000).toUTCString()}; path=/`;
  
      const session = appwriteUser.getSession('current');
      // console.log(session);
      document.cookie = `session=${session.$id}; expires=${new Date(Date.now() + 30 * 24 * 60 * 60 * 1000).toUTCString()}; path=/`;
      const curruser = appwriteUser.get();
      curruser.then((response) => {
        console.log(response);
        document.cookie = `userName=${response.name}; expires=${new Date(Date.now() + 30 * 24 * 60 * 60 * 1000).toUTCString()}; path=/`;
      window.location.href = '/dashboard';
      }, (error) => {
        console.log(error);
        errorMessage = error.message;
      });
    }, (error) => {
      console.log(error);
      errorMessage = error.message;
      appwriteUser.deleteSession('current').then(() => {
        document.cookie = "";
      })
      disabled = email === '' || password === '';
    });
  }
  
</script>
<div class="flex flex-col gap-5 items-center">
  <h1 class="text-white text-4xl text-center font-semibold">
    Log In
  </h1>
  <div>
    <p class="text-red-500 mb-2">{errorMessage}</p>
    <label for="large-input" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">School Email</label>
    <div class="relative mb-6">
      <div class="absolute inset-y-0 start-0 flex items-center ps-3.5 pointer-events-none">
        <svg class="w-4 h-4 text-gray-500 dark:text-gray-400" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 20 16">
            <path d="m10.036 8.278 9.258-7.79A1.979 1.979 0 0 0 18 0H2A1.987 1.987 0 0 0 .641.541l9.395 7.737Z"/>
            <path d="M11.241 9.817c-.36.275-.801.425-1.255.427-.428 0-.845-.138-1.187-.395L0 2.6V14a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V2.5l-8.759 7.317Z"/>
        </svg>
      </div>
      <input bind:value={email} type="text" class={(errorMessage.length ? "border-red-500" : "dark:border-gray-600") + " dark:placeholder-gray-400 dark:text-white dark:bg-gray-700 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-[28rem] ps-10 p-3 dark:focus:ring-blue-500 dark:focus:border-blue-500"} placeholder="name@district.org">
    </div>
    <label for="default-input" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Password</label>
    <input bind:value={password} type="password" class={(errorMessage.length ? "border-red-500" : "dark:border-gray-600") + " dark:placeholder-gray-400 dark:text-white dark:bg-gray-700 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-[28rem] p-3 dark:focus:ring-blue-500 dark:focus:border-blue-500"} placeholder="Password">
  </div>
  <button on:click={login} disabled={disabled} class="text-white disabled:opacity-50 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-10 py-2.5 me-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800">Log In</button>
  <p class="text-center text-gray-300">Don't have an account? <a href="/signup" class="text-blue-500">Sign Up</a></p>
</div>