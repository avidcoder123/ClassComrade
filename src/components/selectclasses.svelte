<script lang="ts">
    import { Query, ID, type Models } from "appwrite";
    import { appwriteDatabases } from "../lib/backend";
    import { COLLECTION, DB_ID } from "../lib/ids";

    export let user: string

    console.log(user)

    let className = "", teacherName = "", acceleration = ""

    let classes: Models.Document[] = []

    let selectedClasses: string[] = []

    let school = ""

    let loading = false

    appwriteDatabases.listDocuments(DB_ID, COLLECTION.Class_User, [
        Query.equal("User", user)
    ]).then(res => res.documents.forEach(doc => {
        selectedClasses = [...selectedClasses, doc.Class]
    }))

    appwriteDatabases.listDocuments(DB_ID, COLLECTION.User_School, [
        Query.equal("User", user)
    ])
    .then(res => res.documents[0])
    .then(doc => {
        school = doc.School
        return appwriteDatabases.listDocuments(DB_ID, COLLECTION.Classes, [
            Query.equal("School", school),
            Query.orderAsc("Name")
        ])
    })
    .then(res => {
        classes = res.documents
    })

    function createClass() {
        let id = ID.unique()
        let name = `${acceleration} ${className} w/ ${teacherName}`
        acceleration = ""
        className = ""
        teacherName = ""
        appwriteDatabases.createDocument(DB_ID, COLLECTION.Classes, id, {
            Name: name,
            School: school
        }).then(d => {
            classes = [...classes, ({
                $id: d.$id,
                Name: name 
            } as unknown as Models.Document)]
            selectedClasses = [...selectedClasses, d.$id]
            updateClasses()
        })
    }

    function updateClasses() {
        loading = true
        appwriteDatabases.listDocuments(DB_ID, COLLECTION.Class_User, [
            Query.equal("User", user)
        ]).then(res => {
            return Promise.all(
                res.documents.map(
                    doc => appwriteDatabases.deleteDocument(DB_ID, COLLECTION.Class_User, doc.$id)
                )
            )
        }).then(() => {
            return Promise.all(selectedClasses.map(c => 
                appwriteDatabases.createDocument(DB_ID, COLLECTION.Class_User, ID.unique(), {
                    Class: c,
                    User: user
                })
            )).then(() => loading = false)
        })
    }
</script>
<div class="flex flex-col items-center gap-4 h-screen">
    <p class="text-4xl text-center font-bold text-white">Select your classes</p>
    <button class="text-white rounded-full bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium text-sm px-6 py-3 me-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800" 
    on:click={() => {
        const modal = document.getElementById('newclassmodal');
        if (modal instanceof HTMLDialogElement && typeof modal.showModal === 'function') {
            modal.showModal();
        }
    }}>+ New Class</button>
    <div class="flex flex-col gap-3">
        {#each classes as c}
            <label class="cursor-pointer label flex-row justify-center">
                <input value={c.$id} bind:group={selectedClasses} on:change={updateClasses} type="checkbox" class="w-6 h-6 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500 dark:focus:ring-blue-600 dark:ring-offset-gray-800 focus:ring-2 dark:bg-gray-700 dark:border-gray-600" />
                <span class="text-lg ms-2 font-medium text-gray-900 dark:text-gray-300">{c.Name}</span>
            </label>
        {/each}    
    </div>
    <button disabled={loading} on:click={() => location.href = "/dashboard"} type="button" class="flex flex-row gap-2 disabled:opacity-50 text-gray-900 bg-white border border-gray-300 focus:outline-none hover:bg-gray-100 focus:ring-4 focus:ring-gray-200 font-medium rounded-full text-sm px-5 py-2.5 me-2 mb-2 dark:bg-gray-800 dark:text-white dark:border-gray-600 enabled:dark:hover:bg-gray-700 dark:hover:border-gray-600 dark:focus:ring-gray-700">
        <svg class="w-3.5 h-3.5 rotate-180 my-auto" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 10">
            <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M1 5h12m0 0L9 1m4 4L9 9"/>
        </svg>
        Back to dashboard
    </button>
</div>
<dialog id="newclassmodal" class="overflow-y-auto overflow-x-hidden fixed top-0 right-0 left-0 z-50 justify-center items-center md:inset-0 bg-white rounded-lg shadow dark:bg-gray-700">
    <form method="dialog" class="modal-box flex flex-col gap-4">
        <div class="flex flex-row">
            <h3 class="font-bold text-2xl text-white text-center">Create Class</h3>
            <button type="button" class="ml-auto text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm w-8 h-8 inline-flex justify-center items-center dark:hover:bg-gray-600 dark:hover:text-white"
            on:click={() => {
                const modal = document.getElementById('newclassmodal');
                if (modal instanceof HTMLDialogElement && typeof modal.close === 'function') {
                    modal.close();
                }
            }}>
              <svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 14">
                  <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6"/>
              </svg>
            </button>
        </div>
      <div class="flex flex-col gap-4 border-t border-t-gray-600 pt-3">
        <label for="database" class="text-white">Class Acceleration</label>
        <select id="database" class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500" bind:value={acceleration}>
            <option value="">Select...</option>
            <option value="On Level">On Level</option>
            <option value="Honors">Honors</option>
            <option value="Dual Credit">Dual Credit</option>
            <option value="AP">AP</option>
            <option value="IB-HL">IB-HL</option>
        </select>
        <label for="name" class="text-white">Class Name (No prefixes)</label>
        <input type="text" bind:value={className} class="dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:bg-gray-700 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-[28rem] p-3 dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="Calculus AB, not AP Calculus AB">
        <label for="name" class="text-white">Professor/Teacher Name</label>
        <input type="text" bind:value={teacherName} class="dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:bg-gray-700 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-[28rem] p-3 dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="Instructor Name">
      </div>
      <button
        class="text-white rounded-full bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium text-sm px-6 py-3 me-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800"
        disabled={!teacherName || !acceleration || !className} on:click={() => {
          createClass()
          const modal = document.getElementById('newclassmodal');
          if (modal instanceof HTMLDialogElement && typeof modal.close === 'function') {
              modal.close();
          }
      }}>Create</button>
    </form>
</dialog>