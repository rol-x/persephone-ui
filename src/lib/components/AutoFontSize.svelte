<script>
  import { onMount } from 'svelte';
  export let text = '';
  
  let container;
  let fontSize = 16;
  let maxSize = 20;

  function resize() {
    if (!container) return;

    const parentWidth = container.parentElement?.offsetWidth || 100;
    maxSize = parentWidth > 100 ? 18 : 12;
    let size = maxSize;
    container.style.fontSize = size + 'px';

    while (container.scrollWidth > parentWidth * 0.95 && size > 9) {
      size -= 1;
      container.style.fontSize = size + 'px';
    }

    fontSize = size;
  }

  onMount(() => {
    resize();
    new ResizeObserver(resize).observe(container.parentElement);
  });
</script>

<span bind:this={container} style="display: inline-block; font-size: {fontSize}px;">
  {text}
</span>

<style>
  span {
    overflow: hidden;
    white-space: nowrap;
  }
</style>
