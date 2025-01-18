<script>
	import DogCard from './DogCard.svelte';
	import '../app.css';
	import { onMount } from 'svelte';

	const baseURL = 'https://frontend-take-home-service.fetch.com';
	let loginName = 'Noah';
	let loginEmail = 'noah.mailloux1@gmail.com';
	let dogObjectArray = $state([]);
	const breedsArray = $state([]);
	let selectedBreed = $state([]);
	let breedSort = $state('asc');

	//Login function
	async function handleLogin() {
		const loginEndpoint = `${baseURL}/auth/login`;
		const response = await fetch(loginEndpoint, {
			method: 'POST',
			credentials: 'include',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify({
				name: loginName,
				email: loginEmail
			})
		});
	}

	//Logout function
	async function handleLogout() {
		const loginEndpoint = `${baseURL}/auth/logout`;
		console.log(
			JSON.stringify({
				name: loginName,
				email: loginEmail
			})
		);
		const response = await fetch(loginEndpoint, {
			method: 'POST',
			credentials: 'include'
		});
	}

	//Get dog breeds
	async function getBreeds() {
		const loginEndpoint = `${baseURL}/dogs/breeds`;
		const response = await fetch(loginEndpoint, {
			credentials: 'include'
		});
		console.log(response);
		const json = await response.json();
		breedsArray.push(...json);
	}

	//
	function getDogSearchEndpoints() {
		//square bracket notation
		//https://frontend-take-home-service.fetch.com/dogs/search?breeds[]=Basset&breeds[]=Beagle
		let endpoint = `/dogs/search?sort=breed:${breedSort}`;

		if (selectedBreed) {
			return `${endpoint}&breeds=${selectedBreed}`;
		}
		return endpoint;
	}

	//Search for dogs
	async function dogSearch() {
		const endpoint = `${baseURL}${getDogSearchEndpoints()}`;
		const response = await fetch(endpoint, {
			credentials: 'include'
		});
		const json = await response.json();
		const result = JSON.stringify(json);
		console.log(response);
		//Now that array of IDs is fetched, fetch dog objects
		fetchDogs(json.resultIds);
	}

	//POST request to fetch dog objects
	async function fetchDogs(dogIdsArray) {
		console.log(dogIdsArray);
		console.log(Array.isArray(dogIdsArray));
		const endpoint = `${baseURL}/dogs`;
		const response = await fetch(endpoint, {
			credentials: 'include',
			method: 'POST',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify(dogIdsArray)
		});
		const json = await response.json();
		dogObjectArray = json;
	}

	function handleBreedCheckbox(breed) {
		if (selectedBreed.includes(breed)) {
			selectedBreed = selectedBreed.filter((item) => item === breed);
		} else {
			selectedBreed.push(breed);
		}
	}
</script>

<h1>Fetch Dog Matcher</h1>
<label>
	<input bind:value={loginName} type="text" required />
</label>
<label>
	<input bind:value={loginEmail} type="email" required />
</label>
<button type="button" onclick={handleLogin}>Login</button>
<button type="button" onclick={handleLogout}>Logout</button>
<button type="button" onclick={getBreeds}>Breeds</button>
<div>Filters</div>
<div>Sort</div>
<div>Sort breed</div>
<select name="Breed Sort" bind:value={breedSort}>
	<option value="asc">Ascending Alphabetically</option>
	<option value="desc">Descending Alphabetically</option>
</select>
<div>Choose Breed</div>

<!-- <select name="Breeds" bind:value={selectedBreed} multiple>
	<option value="">Any Breed</option>
	{#each breedsArray as breed}
		<option value={breed}>{breed}</option>
	{/each}
</select> -->
<div class="breed-select">
	{#each breedsArray as breed}
		<label>
			<input
				type="checkbox"
				name={breed}
				value={breed}
				onclick={() => handleBreedCheckbox(breed)}
			/>
			{breed}
		</label>
	{/each}
</div>
<button type="button" onclick={dogSearch}>Dog Search</button>
<div class="dog-cards">
	<!-- Create a dog card for each object returned by the search results -->
	{#each dogObjectArray as dog}
		<DogCard
			id={dog.id}
			img={dog.img}
			name={dog.name}
			age={dog.age}
			zip_code={dog.zip_code}
			breed={dog.breed}
		/>
	{/each}
</div>
