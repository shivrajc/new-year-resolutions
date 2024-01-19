<script>
  import Counter from './lib/Counter.svelte'
  import MediaQuery from './MediaQuery.svelte';
  import AppDesktop from './lib/AppDesktop.svelte';
  import AppMobile from './lib/AppMobile.svelte';

  const width = 1600;
  const height = 900;
</script>

<main>
  <MediaQuery query="(min-width: 500px)" let:matches>
    {#if matches}
          <AppDesktop {width} {height}/>    
    {/if}
  </MediaQuery>
  
  <MediaQuery query="(max-width: 500px)" let:matches>
    {#if matches}
          <AppMobile width={400} height={1100}/>
    {/if}
  </MediaQuery>         

</main>

