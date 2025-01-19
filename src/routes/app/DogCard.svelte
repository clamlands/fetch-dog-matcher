<script>
	let { id, img, name, age, zip_code, breed, favoritesArray, toggleFavorites } = $props();
</script>

<div
	class="dog-card"
	onclick={() => {
		toggleFavorites(id);
	}}
>
	<img src={img} alt="A dog from search results" />
	<div>Name: {name}</div>
	<div>Age: {age}</div>
	<div>Breed: {breed}</div>
	<div>Zipcode {zip_code}</div>
	{#if favoritesArray.includes(id)}
		<div>Favorite</div>
	{:else}
		<div>Not favorite</div>
	{/if}
</div>
