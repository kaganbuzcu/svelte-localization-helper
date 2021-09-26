<script>
  export let headCells;
  export let bodyRows = [];
</script>

<div class="container">
  <div class="main">
    <table>
      <thead>
        <tr>
          {#each headCells as head}
            <th>{head}</th>
          {/each}
        </tr>
      </thead>
      <tbody>
        {#each bodyRows as bodyElement}
          <tr>
            {#each headCells as head}
              <td>{bodyElement[head] || "-"}</td>
            {/each}
          </tr>
        {/each}
      </tbody>
    </table>
  </div>
</div>

<style>
  .container {
    max-width: 1140px;
    margin: auto;
  }
  table {
    font-family: arial, sans-serif;
    border-collapse: collapse;
    width: 100%;
  }
  td,
  th {
    border: 1px solid #dddddd;
    text-align: left;
    padding: 8px;
  }
</style>
