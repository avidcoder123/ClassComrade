<script lang="ts">
import { appwriteDatabases } from "../../lib/backend";
import { DB_ID, COLLECTION } from "../../lib/ids"
import { ID, type Models, Query } from "appwrite";

export let AuthorName = "";
export let AuthorUid = "";
export let unitID = ""

let SearchAuid = AuthorUid;

let collectionId = "";

let DocumentDesc = "";

let lessons: string[] = []

let lessonName = ""

appwriteDatabases.listDocuments(DB_ID, COLLECTION.CCNote, [
  Query.equal("Unit", unitID),
  Query.orderAsc("Name")
]).then(res => 
  lessons=[...new Set(res.documents.map(x => x.Name))])



function createDocument() {
   appwriteDatabases.createDocument(DB_ID, COLLECTION.CCNote, ID.unique(), {
     AuthorName,
     AuthorUid,
     Name: lessonName,
     Description: DocumentDesc,
     LastUpdated: new Date().toISOString(),
     Unit: unitID
   }).then(res => location.href = `./doc/${res.$id}`)
}

function chooseLesson() {
  if(lessonName == "newlesson") {
    lessonName = ""
    let newName = prompt("Enter lesson name:")
    if(newName && !lessons.map(x => x.toLowerCase()).includes(newName.toLowerCase())) {
      lessons = [...lessons, newName]
      lessonName = newName
    } else {
      alert("Invalid lesson name.")
    }
  }
}
</script>
<div class="flex flex-wrap gap-2">
  <button class="px-5 py-3 text-base font-medium text-center text-white bg-blue-700 rounded-full hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800"
  on:click={() => {
    const modal = document.getElementById('my_modal_1');
    if (modal instanceof HTMLDialogElement && typeof modal.showModal === 'function') {
        modal.showModal();
    }
  }}>
      New Note
  </button>

  <dialog id="my_modal_1" class="overflow-y-auto overflow-x-hidden fixed top-0 right-0 left-0 z-50 justify-center items-center md:inset-0 bg-white rounded-lg shadow dark:bg-gray-700">
    <form method="dialog" class="flex flex-col gap-4">
      <div class="flex flex-row">
        <h3 class="font-bold text-2xl text-white text-center">Create Class</h3>
        <button type="button" class="ml-auto text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm w-8 h-8 inline-flex justify-center items-center dark:hover:bg-gray-600 dark:hover:text-white"
        on:click={() => {
            const modal = document.getElementById('my_modal_1');
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
        <label for="name" class="text-white">Lesson Name</label>
      </div>
      <!--TODO: New lesson-->
      <select bind:value={lessonName} on:change={chooseLesson}
      class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500">
        <option value="">Select Lesson...</option>
        <option value="newlesson">Create New Lesson</option>
        {#each lessons as lesson}
          <option value={lesson}>{lesson}</option>
        {/each}
      </select>
      <label for="description" class="text-white">Notes Title</label>
      <input id="description" bind:value={DocumentDesc} class="dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:bg-gray-700 border text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-[28rem] p-3 dark:focus:ring-blue-500 dark:focus:border-blue-500" placeholder="Description" />
      <button class="text-white rounded-full bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium text-sm px-6 py-3 me-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800"
      disabled={!(lessonName && DocumentDesc)} on:click={() => {
          createDocument();
          const modal = document.getElementById('my_modal_1');
          if (modal instanceof HTMLDialogElement && typeof modal.close === 'function') {
              modal.close();
          }
      }}>Create</button>
    </form>
  </dialog>
</div>
