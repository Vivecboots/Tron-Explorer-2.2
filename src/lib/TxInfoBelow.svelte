<script>
  import { writable, derived } from 'svelte/store';

  let transactionInfo = null;
  let loading = false;
  let searchValue = '';
  let selectedTransaction = null;
  let nextPageUrl = null;
  let previousPageUrl = null;

  const currentTime = writable(Date.now());
  setInterval(() => currentTime.set(Date.now()), 1000);

  const contractAge = derived(
    currentTime,
    ($currentTime, set) => {
      if (selectedTransaction) {
        const ageInSeconds = Math.floor(($currentTime - selectedTransaction.raw_data.timestamp) / 1000);
        const days = Math.floor(ageInSeconds / (3600*24));
        const hours = Math.floor((ageInSeconds % (3600*24)) / 3600);
        const minutes = Math.floor((ageInSeconds % 3600) / 60);
        const seconds = ageInSeconds % 60;
        set({ days, hours, minutes, seconds });
      }
    },
    { days: 0, hours: 0, minutes: 0, seconds: 0 }
  );

  async function fetchTransactions(url) {
    loading = true;
    selectedTransaction = null;

    const options = {
      method: 'GET',
      headers: {
        accept: 'application/json',
        'TRON-PRO-API-KEY': '7a11a8ca-bbbf-43d9-a340-272301ad396b' // Replace with your actual API key
      }
    };

    const response = await fetch(url, options);
    const data = await response.json();

    transactionInfo = data.data;
    nextPageUrl = data.meta.links.next;
    previousPageUrl = data.meta.links.prev;

    loading = false;
  }

  function handleSubmit() {
    const url = `https://api.trongrid.io/v1/accounts/${searchValue}/transactions`;
    fetchTransactions(url);
  }

  function handleNext() {
    if (nextPageUrl) {
      fetchTransactions(nextPageUrl);
    }
  }

  function handlePrevious() {
    if (previousPageUrl) {
      fetchTransactions(previousPageUrl);
    }
  }

  function selectTransaction(transaction) {
    if (selectedTransaction && selectedTransaction.txID === transaction.txID) {
      selectedTransaction = null; // Close the extra information if the same transaction is selected again
    } else {
      selectedTransaction = transaction; // Otherwise, select the new transaction
    }
  }
</script>

<div class="w-9/12 container mx-auto p-4 rounded shadow bg-slate text-white">
  <h2 class="text-2xl font-bold mb-4">Extended Transaction History</h2>

  <div class="flex mb-4">
    <input
      type="text"
      placeholder="Enter Sender Address"
      bind:value={searchValue}
      class="w-[100rem] mr-5 p-3 border border-slate-300 rounded text-black"
    />
    <button on:click|preventDefault={handleSubmit} class="p-2 bg-red-800 text-white rounded">Search</button>
  </div>

  {#if loading}
    <p>Loading...</p>
  {:else if transactionInfo && transactionInfo.length > 0}
    <div class="mt-4">
      <h3 class="text-lg text-red-500 font-bold mb-2">Transaction Details</h3>
      <div>
        {#each transactionInfo as transaction (transaction.txID)}
          <div class="flex mb-2 cursor-pointer justify" on:click={() => selectTransaction(transaction)} on:keydown={() => selectTransaction(transaction)}>
            <div class="font-bold">Transaction ID:</div>
            <div class="ml-2 text-cyan-200">{transaction.txID}</div>
          </div>
          {#if selectedTransaction && selectedTransaction.txID === transaction.txID}
  <div class="ml-4">
    <div class="flex mb-2">
      <div class="font-bold">Completion:</div>
      <div class="ml-2 text-lime-300">{selectedTransaction.ret[0].contractRet}</div>
    </div>
    <div class="flex mb-2">
      <div class="font-bold">Owner Address:</div>
      <div class="ml-2 text-red-400">{selectedTransaction.raw_data.contract[0].parameter.value.owner_address}</div>
    </div>
    <div class="flex mb-2">
      <div class="font-bold">Block Number:</div>
      <div class="ml-2 text-red-400">{selectedTransaction.blockNumber}</div>
    </div>
    <div class="flex mb-2">
      <div class="font-bold">Block Timestamp:</div>
      <div class="ml-2 text-red-400">{selectedTransaction.block_timestamp}</div>
    </div>
    <div class="flex mb-2">
      <div class="font-bold">Expiration:</div>
      <div class="ml-2 text-red-400">{selectedTransaction.raw_data.expiration}</div>
    </div>
    <div class="flex mb-2">
      <div class="font-bold">Reference Block Bytes:</div>
      <div class="ml-2 text-red-400">{selectedTransaction.raw_data.ref_block_bytes}</div>
    </div>
    <div class="flex mb-2">
      <div class="font-bold">Reference Block Hash:</div>
      <div class="ml-2 text-red-400">{selectedTransaction.raw_data.ref_block_hash}</div>
    </div>
    <div class="flex mb-2">
      <div class="font-bold">Timestamp:</div>
      <div class="ml-2 text-red-400">{selectedTransaction.raw_data.timestamp}</div>
    </div>
    <div class="font-bold">Contract Age:</div>
    
    <div>  
  <div class="ml-10 text-lime-400">{$contractAge.days} days {$contractAge.hours} hours {$contractAge.minutes} min {$contractAge.seconds} sec ago</div>
</div>


  </div>
{/if}

        {/each}
      </div>
    </div>
    
    <button on:click={handleNext} disabled={!nextPageUrl} class="p-2 bg-red-800 text-white rounded">Next</button>
  {:else if transactionInfo && transactionInfo.length === 0}
    <p>No transactions found.</p>
  {/if}
</div>
