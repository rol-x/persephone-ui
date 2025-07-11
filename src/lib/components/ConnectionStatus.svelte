<script>
  export let connected = false;
  export let label = '';
</script>

<span title={label} style="display: inline-flex; align-items: center; gap: 0.3rem;">
  <span style="
    width: 0.75rem;
    height: 0.75rem;
    border-radius: 60%;
    background-color: {connected ? 'limegreen' : 'darkred'};
    display: inline-block;
  " />
  <span style="font-size: 0.9rem;">{label}</span>
</span>
