<script lang="ts">
    import { ID, Query } from "appwrite";
    import { appwriteDatabases } from "../../lib/backend";
    import { COLLECTION, DB_ID } from "../../lib/ids";

    export let id: string;
    let addUnit = ''

    let errorMessage = ''

    function createUnit() {
      let attempt = addUnit
        appwriteDatabases.listDocuments(DB_ID, COLLECTION.Units, [
            Query.equal("Class", id)
        ]).then(x => x.documents)
        .then(units => {
            if(!attempt || units.map(u => (u.Name as string).toLowerCase()).includes(attempt.toLowerCase())) {
                errorMessage = "Invalid Unit Name."
            } else {
                return appwriteDatabases.createDocument(DB_ID, COLLECTION.Units, ID.unique(), {
                    Name: attempt,
                    Class: id
                }).then(() => {
                    closeModal()
                    location.reload()
                })
            }
        })
    }

    function create() {
        const modal = document.getElementById('newunitmodal');
        if (modal instanceof HTMLDialogElement && typeof modal.showModal === 'function') {
            modal.showModal();
        }
    }

    const closeModal = () => {
      const modal = document.getElementById('newunitmodal');
      if (modal instanceof HTMLDialogElement && typeof modal.close === 'function') {
          modal.close();
      }
    }
</script>
<button class="px-5 py-3 text-base font-medium text-center text-white bg-blue-700 rounded-full hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
on:click={create}>
    New Unit
</button>
<dialog id="newunitmodal" class="overflow-y-auto overflow-x-hidden fixed top-0 right-0 left-0 z-50 justify-center items-center md:inset-0 bg-white rounded-lg shadow dark:bg-gray-700">
    <div class="flex flex-col gap-4">
      <div class="flex flex-row">
        <h1 class="text-xl text-white">Add Unit</h1>
        <button type="button" class="ml-auto text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm w-8 h-8 inline-flex justify-center items-center dark:hover:bg-gray-600 dark:hover:text-white"
          on:click={closeModal}>
          <svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 14">
              <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6"/>
          </svg>
        </button>
      </div>
      <div class="border-t border-t-gray-600 pt-3">
        {#if errorMessage}
          <span class="text-red-500">{errorMessage}</span>
        {/if}
        <input bind:value={addUnit} type="text" class={(errorMessage.length ? "border-red-500" : "dark:border-gray-600") + " dark:placeholder-gray-400 dark:text-white dark:bg-gray-700 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-[28rem] p-3 dark:focus:ring-blue-500 dark:focus:border-blue-500"} placeholder="Unit Name">
      </div>
      <div class="flex flex-row gap-3">
        <button class="text-white w-full disabled:opacity-50 focus:ring-4 focus:ring-blue-300 font-medium rounded-full text-sm px-10 py-2.5 me-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800"
        disabled={!addUnit.trim().length}
        on:click={() => {
          createUnit()
        }}>
          Create
        </button>
      </div>
    </div>
  </dialog>