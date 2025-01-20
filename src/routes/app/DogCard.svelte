<script>
	let { id, img, name, age, zip_code, breed, favoritesArray, toggleFavorites } = $props();
</script>

<div class="dog-card">
	<div class="img-heart-container">
		<img src={img} alt="A dog from search results" />
		{#if favoritesArray.includes(id)}
			<svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg" class="heart">
				<path
					fill="red"
					stroke="red"
					stroke-width="4"
					d="M 10,30
 A 20,20 0,0,1 50,30
 A 20,20 0,0,1 90,30
 Q 90,60 50,90
 Q 10,60 10,30 z"
				/>
			</svg>
		{:else}
			<svg viewBox="0 0 100 100" xmlns="http://www.w3.org/2000/svg" class="heart">
				<path
					fill="none"
					stroke="red"
					stroke-width="4"
					d="M 10,30
 A 20,20 0,0,1 50,30
 A 20,20 0,0,1 90,30
 Q 90,60 50,90
 Q 10,60 10,30 z"
				/>
			</svg>
		{/if}
	</div>

	<div class="dog-card-fields">
		<div class="dog-left">
			<div>Name: {name}</div>
			<div>Age: {age}</div>
			<div>Breed: {breed}</div>
			<div>ZIP Code: {zip_code}</div>
		</div>
		<div>
			{#if favoritesArray.includes(id)}
				<button
					type="button"
					onclick={() => {
						toggleFavorites(id);
					}}
					class="favorites-button grey-button">Unfavorite</button
				>
			{:else}
				<button
					type="button"
					onclick={() => {
						toggleFavorites(id);
					}}
					class="favorites-button">Favorite</button
				>
			{/if}
		</div>
	</div>
</div>
