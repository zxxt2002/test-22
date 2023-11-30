<script>
    import { createEventDispatcher } from 'svelte';
    import { getDocs, collection } from 'firebase/firestore';
    import { db } from '../../firebaseConfig'; // Make sure to import your db from where it's initialized
    import { historyItems } from '../../historyStore';

    const dispatch = createEventDispatcher();

    export let visible;  // accept the prop from parent
    export let onClose;  // accept the close function from parent

    let selectedContent = ' ';
    let isViewingHistory = true;
    $: items = $historyItems;
    onMount(() => {
        fetchHistory();
    });

    let popupHeight = `${historyItems.length * 50}px`; 
    const viewContent = (item) => {
        selectedContent = item.content; // Display the content of the clicked item
        isViewingHistory = false; // Switch to viewing selected content
    };
    const returnToHistory = () => {
        isViewingHistory = true;
    }
    $: if (!visible) {
        // If the `visible` prop is changed externally, reset local state
        isViewingHistory = true;
        selectedContent = '';
    }
    function closeHistoryPopup() {
        visible = false;
        if (onClose) {
            onClose();
        }
        dispatch('close');
    };
    async function fetchHistory() {
        try {
            const querySnapshot = await getDocs(collection(db, "generatedArticles"));
            const items = querySnapshot.docs.map(doc => {
                let data = doc.data();
                return {
                    id: doc.id,
                    keywords: data.keywords,
                    content: data.sections.join(' '), // Join the sections into a single string
                    timestamp: data.timestamp.toDate().toLocaleString() // Format timestamp
                };
            });
            historyItems.set(items); // Correct way to set the store's value
        } catch (error) {
            console.error("Error fetching history: ", error);
        }
    }
</script>

<!-- Your content before the button -->



{#if visible}
<div class="overlay" on:click={closeHistoryPopup}></div>
<div class="modal" style="--popupHeight: {popupHeight};">
    <button on:click={closeHistoryPopup} class="close-btn">x</button>
    {#if isViewingHistory}
        <h2 class="text-3xl font-medium pb-1">History</h2>
        <div class="history-list">
            {#each items as item}
                <button on:click={() => viewContent(item)} class="history-item">
                    {item.keywords} - {item.timestamp}
                </button>
            {/each}
        </div>
    {:else}
        <button on:click={returnToHistory} class="history-item">Back</button>
        <div class="content">{selectedContent}</div>
    {/if}
</div>
{/if}

<style>
    /* Add the same styles for overlay, modal, and close-btn from your login.svelte */
    .overlay {
        position: fixed;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        background-color: rgba(0, 0, 0, 0.5);
        z-index: 1000; /* Ensure this is high enough to cover the page content */
    }

    .modal {
        position: fixed;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        background: black;
        padding: 20px;
        z-index: 1001; /* Ensure this is higher than the overlay */
        /* Rest of your modal styles */
    }
    .history-list {
        list-style: none;
        padding: 0;
        width: 100%;
        display: flex;
        flex-direction: column;
        gap: 10px; /* spacing between buttons */
    }

    .history-item {
        height: 50px;
    }

    .content {
        font-size: 1rem;
    }

    .close-btn {
        background-color: #383838;
        border: none;
        border-radius: 50%;
        cursor: pointer;
        font-size: 1.2rem;
        width: 30px;
        height: 30px;
        display: flex;
        align-items: center;
        justify-content: center;
        position: absolute;
        top: 10px;
        right: 10px;
    }

</style>
