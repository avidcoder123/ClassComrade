<script lang="ts">
    import { appwriteDatabases } from "../lib/backend";
    import { Query } from "appwrite";
    import { COLLECTION, DB_ID } from "../lib/ids";

    export let assignmentId = '';

    let loadedComments: any[] = [];
    let writtenComment = '';

    $: disabled = !writtenComment;

    async function getComments() {
        let comments = await appwriteDatabases.getDocument(DB_ID, COLLECTION.Assignment, assignmentId)
        loadedComments = comments.Comments
    }

    getComments()

    function addComment() {
        appwriteDatabases.updateDocument(DB_ID, COLLECTION.Assignment, assignmentId, {
            // add comment to comments array
            Comments: [...loadedComments, writtenComment]
        }).then(() => {
            getComments()
            writtenComment = ''
        }
        )
    }
</script>

<main>
    <div class="flex flex-col items-center">
    <h1 class="text-3xl font-bold text-white my-2">Comments</h1>
    <div class="flex flex-col justify-center items-center w-96">
    <div class="flex flex-row gap-6 mb-4">
        <input class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"  bind:value={writtenComment} placeholder="Add a comment" type="text">
        <button class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800" on:click={addComment} {disabled}>Add</button>
    </div>
    <!-- get comments -->
    {#each loadedComments as comment}
        <div class="flex flex-col w-full max-w-[320px] leading-1.5 px-4 py-2 border-gray-200 bg-gray-100 rounded-e-xl rounded-es-xl dark:bg-gray-700">
            <p class="text-sm font-normal py-2.5 text-gray-900 dark:text-white">{comment}</p>
         </div>
    {/each}
    </div>
    </div>
</main>