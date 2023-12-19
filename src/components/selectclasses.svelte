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
<div class="flex flex-col items-center 2xl:scale-125 pt-10 gap-4 h-screen">
    <p class="text-3xl text-center font-bold text-white">Select your classes</p>
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
<dialog id="newclassmodal" class="modal">
    <form method="dialog" class="modal-box bg-gray-200 flex flex-col gap-4">
      <h3 class="font-bold text-2xl text-gray-900 text-center">Create Class</h3>
      <label for="name" class="text-gray-800">Class Name</label>
      <input type="text" id="name" class="input input-bordered bg-gray-400" placeholder="Class Name" bind:value={className} />
      <label for="name" class="text-gray-800">Professor/Teacher Name</label>
      <input type="text" id="name" class="input input-bordered bg-gray-400" placeholder="Professor/Teacher Name" bind:value={teacherName} />
      <label for="database" class="text-gray-800">Class Acceleration</label>
      <select id="database" class="select select-bordered bg-gray-400" bind:value={acceleration}>
          <option value="">Select...</option>
          <option value="On Level">On Level</option>
          <option value="Honors">Honors</option>
          <option value="Dual Credit">Dual Credit</option>
          <option value="AP">AP</option>
          <option value="IB-HL">IB-HL</option>
      </select>
      <button class="btn btn-primary" disabled={!teacherName || !acceleration || !className} on:click={() => {
          createClass()
          const modal = document.getElementById('newclassmodal');
          if (modal instanceof HTMLDialogElement && typeof modal.close === 'function') {
              modal.close();
          }
      }}>Create</button>
      <div class="modal-action">
        <button class="btn btn-error">Cancel</button>
      </div>
    </form>
</dialog>