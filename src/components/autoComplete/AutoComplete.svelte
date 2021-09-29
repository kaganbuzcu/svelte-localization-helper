<script>
  import { createEventDispatcher } from "svelte";
  import { List, ListItem, Subheader, TextField } from "svelte-materialify";

  export let inputDescription = "";
  export let inputLabel = "";
  export let inputPlaceholder = "";
  export let selectedValue;
  export let data = [];
  let filteredData = [];
  let searchText = "";

  const dispatch = createEventDispatcher();

  function onChangeSelectedValue(event) {
    dispatch("changeValue", {
      value: event.target.innerText,
    });
    searchText = "";
  }

  function handleSearch(event) {
    searchText = event.target.value;
    if (data.length > 0) {
      filteredData = data.filter((item) =>
        item.toLowerCase().includes(event.target.value.toLowerCase())
      );
    }
  }
</script>

<div class="input">
  <List class="elevation-2" style="padding: 0px !important; overflow-y">
    <h6 class="mb-3 mt-6">{inputDescription}</h6>
    <TextField
      value={selectedValue}
      on:input={handleSearch}
      placeholder={inputPlaceholder}
      outlined>{inputLabel}</TextField
    >
    <div class="result-wrapper">
      {#if searchText.length > 0}
        {#each filteredData as result}
          <ListItem selectable on:click={onChangeSelectedValue}
            >{result}</ListItem
          >
        {:else}
          <Subheader>Try again :(</Subheader>
        {/each}
      {/if}
    </div>
  </List>
</div>

<style>
  .result-wrapper {
    overflow-y: auto;
    max-height: 200px;
    scrollbar-color: dark;
  }

  .result-wrapper::-webkit-scrollbar {
    width: 9px;
  }

  .result-wrapper::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.3);
    -webkit-border-radius: 3px;
    border-radius: 3px;
  }

  .result-wrapper::-webkit-scrollbar-thumb {
    -webkit-border-radius: 3px;
    border-radius: 3px;
    background: #6200ee;
    -webkit-box-shadow: inset 0 0 6px rgba(0, 0, 0, 0.5);
  }
</style>
