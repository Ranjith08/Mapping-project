<script>
  window.global ||= window;
  import Map from "./Map.svelte";
  import Search from "./Search.svelte";

  let innerWidth;
  let innerHeight;

  let height;

  let map;
  let eventsInHighlight;
  $: height = innerHeight / 1.4;
</script>

<svelte:window bind:innerWidth bind:innerHeight />

<main class="h-screen w-screen">
  <Map bind:map bind:eventsInHighlight />
  <div id="left-bar">
    <div id="meta-info" class="p-4 bg-neutral-800 text-white rounded-xl">
      <h1 class="font-bold font-700 text-3xl">
        Explorer for Ethno-Political activity in India
      </h1>
      <h3>
        <br />
        This is a prototype to explore linked stories of Hindutva, communal violence
        and related incidents. <br /> <br /> Search for incidents or click on an
        event and do a proximity search.
      </h3>
    </div>

    <div id="search" class="mt-3 p-2 w-full bg-neutral-800 rounded-xl">
      <Search bind:map />
    </div>
  </div>

  <div id="right-bar">
    <div id="meta-info" class="p-4 bg-neutral-800 text-white rounded-xl">
      <h2 class="text-xl font-bold font-700 text-3xl">
        Events within a radius of 100km
      </h2>
      <ul class="text-xs">
        {#if eventsInHighlight}
          {#each eventsInHighlight.features as event}
            <li class="py-2">{event.properties.title}</li>
          {/each}
        {/if}
      </ul>
    </div>
  </div>
  <!-- <div id="right-bar" class="p-4">
  <CollapsibleCard>
      <h2 class="font-semibold text-xl " slot="header">Events in focus &#8964;</h2>
      <div slot='body'>
        <SelectedFeatures 
          height={height}/>
      </div>

  </CollapsibleCard>
</div> -->
</main>

<style>
  #right-bar {
    @apply max-h-[70vh] overflow-y-auto;
  }
  #left-bar {
    position: absolute;
    top: 3%;
    left: 3%;
    width: 50%;
  }

  #right-bar {
    position: absolute;
    top: 5%;
    right: 5%;
    width: 90%;
    max-width: 20% !important;
  }
  #left-bar,
  #right-bar {
    z-index: 2000;
    text-align: left;
    color: black;
    border-radius: 10px;

    max-width: 20%;
  }

  #meta-info {
  }
</style>
