<script>
import EditorJS from '@editorjs/editorjs';
// only import the following: Embed, header, link, list, quote, table, InlineCode, warning, paragraph
import List from '@editorjs/list';
import Quote from '@editorjs/quote';
import Table from '@editorjs/table';
import InlineCode from '@editorjs/inline-code';
import Warning from '@editorjs/warning';
import Paragraph from 'editorjs-paragraph-with-alignment';
import Header from 'editorjs-header-with-alignment'
import ToggleBlock from 'editorjs-toggle-block';
import Hyperlink from 'editorjs-hyperlink';
import Marker from '@editorjs/marker';
import ImageTool from '@editorjs/image';
import DragDrop from 'editorjs-drag-drop';
import Undo from 'editorjs-undo';
import Underline from '@editorjs/underline';
import NestedList from '@editorjs/nested-list';
import ChangeCase from 'editorjs-change-case';
import Strikethrough from '@sotaproject/strikethrough';
import editorjsCodeflask from '@calumk/editorjs-codeflask';
import edjsHTML from 'editorjs-html';
import * as Backend from '../lib/backend';
import * as BackendID from '../lib/ids';
import { ID,Query } from 'appwrite';
    import { not_equal } from 'svelte/internal';
    import { findLastIndex } from 'lodash';

export let pid = "";
export let user = "";
export let name = "";
let databaseId = BackendID.DB_ID;
let collectionId = BackendID.COLLECTION.CCNote;

let LoadedTitle = "";
let loadedData = '';
let EdJsCleanData = "";
let LoadedDate = "";
let docFileLocation = "";
let docVisibility = null;
let AuthorUid = "";

let canRead = false;

let editor;
let ParsedHTML = []


Backend.appwriteDatabases.getDocument(databaseId, collectionId, pid).then((response) => {
    console.log(response);
    if (response.Content !== ' ' || response !== null || response !== undefined) {
        loadedData = response.Content;
        LoadedTitle = response.Description;
        LoadedDate = response.LastUpdated;
        docVisibility = true;
        AuthorUid = response.AuthorUid;
        console.log('The document is not empty' + loadedData);
        EdJsCleanData = response
        if (loadedData === ' ') {
            loadedData = JSON.stringify(editor.save());
        }
        if (user == AuthorUid) {
            canRead = true;
        } else {
            canRead = false;
        }
    } else {
        loadedData = editor.save();
        console.log('The document is empty' + loadedData);
    }

    editor = new EditorJS(
        {
            holder: 'editor',
            placeholder: 'Start writing your article... Press tab to see the toolbar',
            readOnly: !canRead,
            tools: {
                header: Header,
                list: {
                    class: List,
                    inlineToolbar: true,
                    config: {
                        defaultStyle: 'unordered'
                    },
                },
                quote: Quote,
                table: Table,
                inlineCode: InlineCode,
                warning: Warning,
                code: editorjsCodeflask,
                underline: Underline,
                changeCase: ChangeCase,
                strikethrough: Strikethrough,
                image: {
                    class : ImageTool,
                        config: {
                            uploader: {uploadByFile}
                        }
                },
                marker: {
                    class: Marker,
                    inlineToolbar: true,
                    shortcut: 'CMD+SHIFT+M',
                },
                paragraph: {
                    class: Paragraph,
                    inlineToolbar: true,
                },
                toggleBlock: {
                    class: ToggleBlock,
                    inlineToolbar: true,
                },
                hyperlink: {
                    class: Hyperlink,
                    inlineToolbar: true,
                },
            },
            data: JSON.parse(loadedData),
        }
    );



// onready function that is called when the editor is ready
editor.isReady.then(() => {
    console.log('Editor.js is ready to work!');
    new Undo({ editor });
    new DragDrop(editor);
    const edjsParser = edjsHTML();
    ParsedHTML = edjsParser.parse(JSON.parse(loadedData));
    // returns array of ParsedHTML strings per block
    console.log(ParsedHTML);
});

function uploadByFile(file) {
 return Backend.appwriteStorage.createFile('64ada07bbca91d21cdbc', ID.unique(), file).then((res) => {
   return {
    success: 1,
    file   : {
     url: `https://cloud.appwrite.io/v1/storage/buckets/64ada07bbca91d21cdbc/files/${res.$id}/view?project=648bc5020fbda818412e`,
    }
   }
  });
}

    document.getElementById('save').onclick = function save() {
        editor.save().then((outputData) => {
            Backend.appwriteDatabases.updateDocument(databaseId, collectionId, pid,
            {
                Content: JSON.stringify(outputData),
                Description: LoadedTitle,
                LastUpdated: new Date().toISOString()
            }
            ).then((response) => {
            }, (error) => {
                console.log(error);
            });
        }).catch((error) => {
            console.log('Saving failed: ', error)
        });
    }
}, (error) => {
    console.log(error);
});

// autosave function that is called every 5 seconds or when the user leaves the page
function autoSave() {
    editor.save().then((outputData) => {
        Backend.appwriteDatabases.updateDocument(databaseId, collectionId, pid,
        {
            Content: JSON.stringify(outputData),
            Description: LoadedTitle,
            LastUpdated: new Date().toISOString(),
        }
        ).then((response) => {
        }, (error) => {
            console.log(error);
        });
    }).catch((error) => {
        console.log('Saving failed: ', error)
    });
}

function deleteDoc() {
    Backend.appwriteDatabases.deleteDocument(databaseId, collectionId, pid).then((response) => {
        console.log(response);
        window.location.href = '/dashboard';
    }, (error) => {
        console.log(error);
    });
}

// call the autosave function every 5 seconds
setInterval(autoSave, 5000);

// call the autosave function when the user leaves the page
window.onbeforeunload = function () {
    autoSave();
}

// a function to make a clone of the document
// function clone() {
//     Backend.appwriteDatabases.createDocument(
//         BackendID.DB_ID,
//         BackendID.COLLECTION.Notes,
//         ID.unique(),
//         {
//             Content: loadedData,
//             Name: LoadedTitle,
//             AuthorName: name,
//             AuthorUid: user,
//             IsPublic: docVisibility,
//         }
//     ).then((response) => {
//         console.log(response);
//         window.location.href = '/doc/'+response.$id;
//     }, (error) => {
//         console.log(error);
//     });
// }

const closeModal = () => {
      const modal = document.getElementById('settingModal');
      if (modal instanceof HTMLDialogElement && typeof modal.close === 'function') {
          modal.close();
      }
    }
</script>
<main>
    <div class="min-h-screen">
        {#if canRead}
            {#if docVisibility==null}=
                <div class=" flex flex-col justify-center items-center pt-[40vh] gap-20">
                <p class="text-center text-6xl text-slate-950 mx-auto w-[75vw]">🌐</p>
                <span class="loading loading-infinity loading-lg text-black mx-auto"></span>
                </div>
                {:else}
                {#if docVisibility==true || user === AuthorUid}
                <div class=" flex flex-row justify-center items-center gap-8 p-2">
                <input type="text" bind:value={LoadedTitle} placeholder="Title" class="border-none text-4xl font-bold text-center bg-transparent active:border-none mx-auto self-center h-16 text-white min-w-96">
                <p class="text-center text-gray-400 text-sm pr-8">Last updated: {LoadedDate.substring(0, 10).replaceAll('-', '/') + ' ' + LoadedDate.substring(11, 16).replaceAll('-', ':')}</p>
                <!--  checkboxes for public and private -->
                <div class="flex flex-row justify-center items-center gap-2">
                <label class="flex items-center">
                {#if user === AuthorUid}
                    <!-- Open the modal using ID.showModal() method -->
                    <button class="text-white bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2 dark:bg-blue-600 dark:hover:bg-blue-700 focus:outline-none dark:focus:ring-blue-800" onclick="settingModal.showModal()">Settings</button>
                    <dialog id="settingModal" class="overflow-y-auto overflow-x-hidden fixed top-0 right-0 left-0 z-50 justify-center items-center md:inset-0 bg-white rounded-lg shadow dark:bg-gray-700">
                        <div class="flex flex-col">
                          <div class="flex flex-row justify-end w-full">
                            <h1 class="text-xl text-white">Settings</h1>
                            <button type="button" class="ml-auto mr-0 text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm w-8 h-8 justify-center items-center dark:hover:bg-gray-600 dark:hover:text-white"
                              on:click={closeModal}>
                              <svg class="w-3 h-3 mx-auto" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 14 14">
                                  <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6"/>
                              </svg>
                            </button>
                          </div>
                          <div class="border-t border-t-gray-600 pt-3">
                            <div class="modal-action flex justify-center items-center flex-row gap-3">
                                <button class="focus:outline-none text-white bg-red-700 hover:bg-red-800 focus:ring-4 focus:ring-red-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2 dark:bg-red-600 dark:hover:bg-red-700 dark:focus:ring-red-900" on:click={deleteDoc}>Delete</button>
                                <button  class="focus:outline-none text-white bg-green-700 hover:bg-green-800 focus:ring-4 focus:ring-green-300 font-medium rounded-lg text-sm px-5 py-2.5 me-2 mb-2 dark:bg-green-600 dark:hover:bg-green-700 dark:focus:ring-green-800" on:click={() => {navigator.clipboard.writeText(window.location.href)}}>Copy Share Link</button>
                            </div>
                          </div>
                        </div>
                      </dialog>
        
                    {:else}
                    <!-- {#if pid!=""}
                        <button class="btn text-gray-800 bg-gray-200 gap-2 mx-auto z-[100001]  hover:bg-gray-300" on:click={clone}>Clone</button>
                    {/if} -->
                {/if}
                </label>
                </div>
                </div>
                <div>
                    <div id="editor"></div>
                </div>
                {:else}
                    <div class=" flex flex-col justify-center items-center pt-[40vh] gap-20 mx-auto">
                        <p class="text-center text-6xl text-slate-950 mx-auto w-[75vw]">🔒</p>
                        <p class="text-center text-2xl text-slate-950 mx-auto w-[75vw]">You do not have access to this document</p>
                    </div>
                {/if}
            {/if}
        {:else}
            <div class="flex flex-row justify-center items-center gap-8 p-2">
                <input type="text" bind:value={LoadedTitle} placeholder="Title" class="border-none text-4xl font-bold text-center bg-transparent active:border-none mx-auto self-center h-16 text-white">
                <p class="text-center text-gray-400 text-sm pr-8">Last updated: {LoadedDate.substring(0, 10).replaceAll('-', '/') + ' ' + LoadedDate.substring(11, 16).replaceAll('-', ':')}</p>
                <!--  checkboxes for public and private -->
            </div>
            <!-- <div class="flex flex-col items-center p-2">
                <div class="w-full px-80">
                    {#each ParsedHTML as line}
                        {@html line}
                    {/each}
                </div>
            </div> -->
        
            <div id="editor"></div>
        {/if}
    </div>

<style>
    .cdx-block, .ce-header, .ce-toolbar__plus, .ce-toolbar__settings-btn, .ce-popover-item__title {
        color: #ffffff;
    }

    .cdx-search-field, .ce-popover__search, .cdx-settings-button  {
        background-color: #ffffff;
    }

    div:has(> .cdx-settings-button) {
        display: flex;
        flex-direction: row;
        gap: 5px;
    }

    .tc-toolbox__toggler>svg>rect {
        fill: #111827;
    }

    .tc-toolbox__toggler>svg>rect:hover {
        fill: #1F2937;
    }

    .tc-toolbox__toggler>svg>circle {
        fill: #FFFFFF;
    }

    .ce-popover-item:hover {
        background-color: #374151 !important;
    }
    .ce-block--selected>div>div, .tc-cell--selected, .tc-cell--selected:hover, .tc-add-row:hover, .tc-add-column:hover {
        background-color: #374151 !important;
    }
    .cdx-warning::before {
        background-color: #FFFFFF;
        border-radius: 5px;
    }
    .tc-add-row:hover::before {
        background-color: #374151 !important;
    }

    .ce-popover, .tc-popover {
        background-color: #1F2937 !important;
    }

    .ce-toolbar__plus:hover, .ce-toolbar__settings-btn:hover {
        background-color: #1F2937 !important;
    }

    h1 {
        font-size: 2rem;
    }
    h2 {
        font-size: 1.5rem;
    }
    h3 {
        font-size: 1.17rem;
    }
    h4 {
        font-size: 1rem;
    }
    h5 {
        font-size: .83rem;
    }
    h6 {
        font-size: .67rem;
    }

    /*  remove border from input */
    input {
        border: none;
        outline: none;
        /* center the input */
        display: block;
        margin: 0 auto;
        padding-top: 1rem;
    }
    /*  make the barriers background color transparent */
    #barrier {
        background-color: transparent;
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index: 1000;
    }
</style>
</main>
