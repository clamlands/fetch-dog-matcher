<script>
	import DogCard from './DogCard.svelte';
	import '../../app.css';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';

	const baseURL = 'https://frontend-take-home-service.fetch.com';
	let dogIdsArray = $state([]);
	let dogObjectArray = $state([]);
	let numberOfPages = $derived(Math.ceil(dogObjectArray.length / 25));
	let currentPage = $state(1);
	let currentPageDogObjectArray = $derived(
		dogObjectArray.slice((currentPage - 1) * 25, currentPage * 25 + 1)
	);
	// let currentPageDogObjectArray = $derived(dogObjectArray);
	const breedsArray = $state([]);
	let selectedBreed = $state([]);
	let searchSort = $state('breed-asc');
	let ageMin = $state(0);
	let ageMax = $state(20);
	let favoritesArray = $state([]);
	let matchObject = $state();

	//Automatically update search results upon changing any sorting or breed selection
	$effect(() => {
		fetchDogIds();
	});

	onMount(() => {
		getBreeds();
	});

	//Logout function
	async function handleLogout() {
		const loginEndpoint = `${baseURL}/auth/logout`;
		const response = await fetch(loginEndpoint, {
			method: 'POST',
			credentials: 'include'
		});
		if (response.ok) {
			goto('./');
		}
	}

	//Get dog breeds
	async function getBreeds() {
		const loginEndpoint = `${baseURL}/dogs/breeds`;
		const response = await fetch(loginEndpoint, {
			credentials: 'include'
		});
		const json = await response.json();
		breedsArray.push(...json);
	}

	//
	function getDogSearchEndpoints() {
		//square bracket notation
		//https://frontend-take-home-service.fetch.com/dogs/search?breeds[]=Basset&breeds[]=Beagle

		let endpoint = `/dogs/search${getSearchSortEndpoints()}&ageMin=${ageMin}&ageMax=${ageMax}`;

		//adds endpoints for all breed selections
		if (Array.isArray(selectedBreed)) {
			selectedBreed.forEach((breed) => {
				endpoint = endpoint + `&breeds[]=${breed}`;
			});
		}
		return endpoint;
	}

	//handles logic for deciding what search sort endpoint is to be used
	function getSearchSortEndpoints() {
		if (searchSort === 'breed-asc') {
			return '?sort=breed:asc';
		} else if (searchSort === 'breed-desc') {
			return '?sort=breed:desc';
		} else if (searchSort === 'name-asc') {
			return '?sort=name:asc';
		} else if (searchSort === 'name-desc') {
			return '?sort=name:desc';
		} else if (searchSort === 'age-asc') {
			return '?sort=age:asc';
		} else return '?sort=age:desc';
	}

	//Search for dogs
	async function fetchDogIds() {
		const tempDogIdsArray = [];
		let endpoint = getDogSearchEndpoints();
		let nextExists;
		do {
			if (tempDogIdsArray.length >= 100) {
				break;
			}
			const response = await fetch(`${baseURL}${endpoint}`, {
				credentials: 'include'
			});
			const json = await response.json();
			if ('next' in json) {
				nextExists = true;
			} else {
				nextExists = false;
			}
			//update the endpoint to get next 25 results
			endpoint = json.next;
			tempDogIdsArray.push(...json.resultIds);
		} while (nextExists);

		// const response = await fetch(endpoint, {
		// 	credentials: 'include'
		// });
		// const json = await response.json();

		// console.log(json);
		// dogIdsArray.push(json.resultIds);
		//Now that array of IDs is fetched, fetch dog objects
		dogIdsArray.push(...tempDogIdsArray);
		fetchDogs(tempDogIdsArray);
	}

	//POST request to fetch dog objects
	async function fetchDogs(dogIdsArray) {
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
			selectedBreed = selectedBreed.filter((item) => item !== breed);
		} else {
			selectedBreed.push(breed);
		}
	}

	//Checks the Id of each dog card with the favoritesArray to display whether it is favorited
	function toggleFavorites(id) {
		if (favoritesArray.includes(id)) {
			favoritesArray = favoritesArray.filter((item) => item !== id);
		} else {
			favoritesArray.push(id);
		}
	}

	//submits the favorited dogs to get the ID for the dog match
	async function getMatchId() {
		const endpoint = `${baseURL}/dogs/match`;
		const response = await fetch(endpoint, {
			credentials: 'include',
			method: 'POST',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify(favoritesArray)
		});
		const json = await response.json();
		const matchId = json.match;
		fetchMatchObject([matchId]);
	}

	//POST request to fetch dog objects
	async function fetchMatchObject(matchId) {
		const endpoint = `${baseURL}/dogs`;
		const response = await fetch(endpoint, {
			credentials: 'include',
			method: 'POST',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify(matchId)
		});
		const json = await response.json();
		matchObject = json[0];
	}
</script>

<section class="top-section">
	<h1>Fetch Dog Matcher</h1>
	<div>Find the right dog for you!</div>
	<p class="instructions">
		Choose your favorite breeds and filter your search by age and location.
	</p>
	<hr />
	<div class="align-left">Select Breeds:</div>
	<div class="breed-select margin-bottom-small">
		{#each breedsArray as breed}
			<label class="breeds">
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
	<label class="filter">
		Sort by:
		<select name="Breed Sort" bind:value={searchSort}>
			<option value="breed-asc">Breed - A to Z</option>
			<option value="breed-desc">Breed - Z to A</option>
			<option value="name-asc">Name - A to Z</option>
			<option value="name-desc">Name - Z to A</option>
			<option value="age-asc">Age - Young to Old</option>
			<option value="age-desc">Age - Old to Young</option>
		</select>
	</label>

	<div class="filter">
		Age Range:
		<label>
			Min
			<select name="Minimum Age" bind:value={ageMin}>
				{#each { length: 21 }, index}
					<option value={index}>{index}</option>
				{/each}
			</select>
		</label>

		<label>
			Max
			<select name="Maximum Age" bind:value={ageMax}>
				{#each { length: 21 }, index}
					<option value={index}>{index}</option>
				{/each}
			</select>
		</label>
	</div>
	<p class="instructions">
		As you search, click the images of the dogs you love most, and they will be added to your
		favorites. Once you're finished, click the "Get Your Match!" button and we'll pick the best dog
		for you.
	</p>
</section>

<section class="bottom-section">
	<button type="button" onclick={getMatchId} class="big-button">Get Your Match!</button>
	<div class="page-numbers">
		{#each { length: numberOfPages }, index}
			<button
				onclick={() => {
					currentPage = index + 1;
				}}
				class={index + 1 === currentPage ? 'page-number active' : 'page-number'}>{index + 1}</button
			>
		{/each}
	</div>
	{#if matchObject}
		<div class="dog-card match-card">
			<h1>YOUR MATCH!!!</h1>
			<img src={matchObject.img} alt="Dog match" />
			<div>Name: {matchObject.name}</div>
			<div>Age: {matchObject.age}</div>
			<div>Breed: {matchObject.breed}</div>
			<div>Zipcode {matchObject.zip_code}</div>
		</div>
	{/if}
	<div class="dog-cards">
		<!-- Create a dog card for each object returned by the search results -->
		{#each currentPageDogObjectArray as dog}
			<DogCard
				id={dog.id}
				img={dog.img}
				name={dog.name}
				age={dog.age}
				zip_code={dog.zip_code}
				breed={dog.breed}
				{favoritesArray}
				{toggleFavorites}
			/>
		{/each}
	</div>
	<div class="page-numbers">
		{#each { length: numberOfPages }, index}
			<button
				onclick={() => {
					currentPage = index + 1;
				}}
				class={index + 1 === currentPage ? 'page-number active' : 'page-number'}>{index + 1}</button
			>
		{/each}
	</div>
	<button type="button" onclick={handleLogout} class="big-button">Logout</button>
</section>
