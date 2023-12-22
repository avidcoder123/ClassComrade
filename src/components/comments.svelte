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
    <div class="flex flex-col justify-center items-center">
    <h1 class="text-3xl font-bold">Comments</h1>
    <div class="flex flex-col justify-center items-center w-96">
    <div class="flex-row gap-6 pb-4">
        <input class="input input-bordered input-info w-72 mt-2" bind:value={writtenComment} placeholder="Add a comment" type="text">
        <button class="btn btn-success disabled:btn-error"on:click={addComment} {disabled}>Add</button>
    </div>
    <!-- get comments -->
    {#each loadedComments as comment}
        <div class="chat chat-start w-full">
            <div class="chat-bubble chat-bubble-secondary">{comment}</div>     
        </div>
    {/each}
    </div>
    </div>
</main>