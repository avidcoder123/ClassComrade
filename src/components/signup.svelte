<script lang="ts">
  import {appwriteUser} from '../lib/backend';
  import { appwriteDatabases } from "../lib/backend";
  import { Query } from "appwrite";
  import { COLLECTION, DB_ID } from "../lib/ids";
  import { ID } from 'appwrite';
  import type { Models } from "appwrite/types"
  
  let email = '';
  let password = '';
  let confirmPassword = '';
  let userName = '';
  let errorMessage = '';
  let school = '';

  let addSchool = '';

  $: disabled = !email || !password || !userName || !confirmPassword;

  $: emailDomain = email.split("@")[1]

  let schools: Models.Document[] = []

  function updateSchools() {
    let domain = emailDomain.toLowerCase().trim()
    console.log(domain)
    if(
      domain == "gmail.com" ||
      domain == "protonmail.com" ||
      domain == "yahoo.com" ||
      domain == "hotmail.com") {
        errorMessage = "You must use your school email!"
        email = ""
    }
    school = ''
    appwriteDatabases.listDocuments(DB_ID, COLLECTION.Schools, [
      Query.equal("Domain", domain)
    ]).then(res => {schools = res.documents})
    
  }

  function createSchool() {
    let domain = emailDomain.toLowerCase().trim()
    appwriteDatabases.createDocument(DB_ID, COLLECTION.Schools, ID.unique(), {
      "Name": addSchool,
      "Domain": domain
    }).then(res => {
      updateSchools()
      school = res.$id
    })
  }

  function checkForNew() {
    if(school == "newschool") {
      school=""
        const modal = document.getElementById('newschoolmodal');
        if (modal instanceof HTMLDialogElement && typeof modal.showModal === 'function') {
            modal.showModal();
        }
    }
  }

  function SignUp() {
    if(confirmPassword !== password) {
      errorMessage = "Passwords do not match."
      return
    }
    const promise = appwriteUser.create( ID.unique() , email, password, userName);
  
    promise.then((response) => {
      appwriteDatabases.createDocument(DB_ID, COLLECTION.User_School, ID.unique(), {
        User: response.$id,
        School: school
      }).then(() => window.location.href = '/login')
    }, (error) => {
      console.log(error);
      if(error.message == "A user with the same id, email, or phone already exists in this project.") {
        appwriteUser.createEmailSession(email, password).then(res => {
          appwriteDatabases.listDocuments(DB_ID, COLLECTION.User_School, [Query.equal("User", res.$id)])
          .then(r => r.documents)
          .then(x => {
            if(x.length > 0) {
              errorMessage = error.message;
            } else {
              appwriteDatabases.createDocument(DB_ID, COLLECTION.User_School, ID.unique(), {
                User: res.userId,
                School: school
              }).then(() => window.location.href = '/login')
            }
          })
        })
      } else {
        errorMessage = error.message;
      }
    });
  }
</script>
<div class="flex flex-col gap-5 items-center h-screen">
  <h1 class="text-white text-4xl text-center font-semibold">
    Sign Up
  </h1>
  <div class="flex flex-col gap-6">
    <p class="text-red-500 mb-2">{errorMessage}</p>
    <div>
      <label for="large-input" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">Name</label>
      <div class="relative">
        <div class="absolute inset-y-0 start-0 flex items-center ps-3.5 pointer-events-none">
          <svg class="w-4 h-4 text-gray-500 dark:text-gray-400" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 20 20">
            <path d="M10 0a10 10 0 1 0 10 10A10.011 10.011 0 0 0 10 0Zm0 5a3 3 0 1 1 0 6 3 3 0 0 1 0-6Zm0 13a8.949 8.949 0 0 1-4.951-1.488A3.987 3.987 0 0 1 9 13h2a3.987 3.987 0 0 1 3.951 3.512A8.949 8.949 0 0 1 10 18Z"/>
          </svg>
        </div>
        <input bind:value={userName} type="text" class={(errorMessage.length ? "border-red-500" : "dark:border-gray-600") + " dark:placeholder-gray-400 dark:text-white dark:bg-gray-700 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-[28rem] ps-10 p-3 dark:focus:ring-blue-500 dark:focus:border-blue-500"} placeholder="John Doe">
      </div>
    </div>
    <div>
      <label for="large-input" class="block mb-2 text-sm font-medium text-gray-900 dark:text-white">School Email</label>
      <div class="relative">
        <div class="absolute inset-y-0 start-0 flex items-center ps-3.5 pointer-events-none">
          <svg class="w-4 h-4 text-gray-500 dark:text-gray-400" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 20 16">
              <path d="m10.036 8.278 9.258-7.79A1.979 1.979 0 0 0 18 0H2A1.987 1.987 0 0 0 .641.541l9.395 7.737Z"/>
              <path d="M11.241 9.817c-.36.275-.801.425-1.255.427-.428 0-.845-.138-1.187-.395L0 2.6V14a2 2 0 0 0 2 2h16a2 2 0 0 0 2-2V2.5l-8.759 7.317Z"/>
          </svg>
        </div>
        <input bind:value={email} on:change={updateSchools} type="text" class={(errorMessage.length ? "border-red-500" : "dark:border-gray-600") + " dark:placeholder-gray-400 dark:text-white dark:bg-gray-700 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-[28rem] ps-10 p-3 dark:focus:ring-blue-500 dark:focus:border-blue-500"} placeholder="name@district.org">
      </div>
    </div>
    <div>
      <label for="default-input" class="block text-sm font-medium text-gray-900 dark:text-white">Password</label>
      <input bind:value={password} type="password" class={(errorMessage.length ? "border-red-500" : "dark:border-gray-600") + " dark:placeholder-gray-400 dark:text-white dark:bg-gray-700 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-[28rem] p-3 dark:focus:ring-blue-500 dark:focus:border-blue-500"} placeholder="Password">
    </div>
    <div>
      <label for="default-input" class="block text-sm font-medium text-gray-900 dark:text-white">Confirm Password</label>
      <input bind:value={confirmPassword} type="password" class={(errorMessage.length ? "border-red-500" : "dark:border-gray-600") + " dark:placeholder-gray-400 dark:text-white dark:bg-gray-700 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-[28rem] p-3 dark:focus:ring-blue-500 dark:focus:border-blue-500"} placeholder="Confirm Password">
    </div>
    <select class="rounded-md dark:text-white dark:bg-gray-700 dark:border-gray-600 p-3" bind:value={school} on:change={checkForNew} disabled={!(email.split("@")[0] && emailDomain)}>
      <option value="">Select school...</option>
      {#if schools.length}
        {#each schools as school}
          <option value={school.$id}>{school.Name}</option>
        {/each}
      {/if}
      <option value="newschool">Add new school</option>
    </select>
  </div>
  <button on:click={SignUp} disabled={disabled} class="text-white disabled:opacity-50 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-10 py-2.5 me-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800">Sign Up</button>
  <p class="text-center text-gray-300">Already have an account? <a href="/login" class="text-blue-500">Sign Up</a></p>
</div>
<dialog id="newschoolmodal" class="overflow-y-auto overflow-x-hidden fixed top-0 right-0 left-0 z-50 justify-center items-center md:inset-0 bg-white rounded-lg shadow dark:bg-gray-700">
  <div class="flex flex-col gap-4">
    <div class="flex flex-row">
      <h1 class="text-xl text-white">Add School</h1>
      <button type="button" class="ml-auto text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm w-8 h-8 inline-flex justify-center items-center dark:hover:bg-gray-600 dark:hover:text-white"
        on:click={() => {
            const modal = document.getElementById('newschoolmodal');
            if (modal instanceof HTMLDialogElement && typeof modal.close === 'function') {
                modal.close();
            }
        }}>
        <svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 14">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6"/>
        </svg>
      </button>
    </div>
    <div class="border-t border-t-gray-600 pt-3">
      <input bind:value={addSchool} type="text" class={(errorMessage.length ? "border-red-500" : "dark:border-gray-600") + " dark:placeholder-gray-400 dark:text-white dark:bg-gray-700 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-[28rem] p-3 dark:focus:ring-blue-500 dark:focus:border-blue-500"} placeholder="School Name">
    </div>
    <div class="flex flex-row gap-3">
      <button class="text-white w-full disabled:opacity-50 focus:ring-4 focus:ring-blue-300 font-medium rounded-full text-sm px-10 py-2.5 me-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800"
      disabled={!addSchool.trim().length}
      on:click={() => {
        createSchool()
        const modal = document.getElementById('newschoolmodal');
        if (modal instanceof HTMLDialogElement && typeof modal.close === 'function') {
            modal.close();
        }
    }}>
        Create
      </button>
    </div>
  </div>
</dialog>