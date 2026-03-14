<script lang="ts">
</script>

<slot />

<style>
  :global(*) {
    box-sizing: border-box;
    margin: 0;
    padding: 0;
  }

  :global(body) {
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, sans-serif;
    background: #f8fafc;
    color: #1e293b;
    min-height: 100vh;
  }

  :global(input, select, textarea) {
    font-family: inherit;
  }
</style>
