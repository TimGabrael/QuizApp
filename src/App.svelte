<script>
import MultipleChoice from "./MultipleChoice.svelte";

let views = [MultipleChoice]

let viewportComponent = null;
let viewportComponentElement = null;
let currentView = 0;

function OnMainClick()
{
	if(viewportComponentElement != null)
	{
		viewportComponentElement.OnClickAnywhere();	
	}
}

function UpdateViewportComponent(index)
{
	currentView = index % views.length;
	viewportComponent = views[index];
}
UpdateViewportComponent(0);

</script>

<svelte:window  on:click={OnMainClick}/>
<main>

{#if viewportComponent == views[currentView]}
	<svelte:component this={viewportComponent} bind:this={viewportComponentElement}></svelte:component>
{/if}
</main>

<style>
	:global(body){
		background-color: #202124;
		color: antiquewhite;
	}
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>