<script lang="ts">
	import DogCard from './DogCard.svelte';
	import '../../app.css';
	import { goto } from '$app/navigation';
	import { onMount } from 'svelte';

	const baseURL = 'https://frontend-take-home-service.fetch.com';
	let dogIdsArray: string[] = $state([]);
	let dogObjectArray = $state([]);
	let numberOfPages = $derived(Math.ceil(dogObjectArray.length / 25));
	let currentPage = $state(1);
	//display only 25 results per page
	let currentPageDogObjectArray: Dog[] = $derived(
		dogObjectArray.slice((currentPage - 1) * 25, currentPage * 25 + 1)
	);
	const breedsArray: string[] = $state([]);
	let selectedBreed: string[] = $state([]);
	let searchSort = $state('breed-asc');
	let ageMin = $state(0);
	let ageMax = $state(20);
	let favoritesArray: string[] = $state([]);
	let matchObject: Dog | undefined = $state();
	let matchLocationObject: Location | undefined = $state();
	let favoritesFlag: boolean = $state(false);

	//references to "Get Your Match!" button to be scrolled to
	let scrollToMatchRef: HTMLElement;

	//Automatically update search results upon changing any sorting/filtering/breed selection
	$effect(() => {
		fetchDogIds();
	});

	onMount(() => {
		getBreeds();
	});

	// Data models for Dog and Location objects return by the API
	interface Dog {
		id: string;
		img: string;
		name: string;
		age: number;
		zip_code: string;
		breed: string;
	}

	interface Location {
		zip_code: string;
		latitude: number;
		longitude: number;
		city: string;
		state: string;
		county: string;
	}

	//Logout function
	async function handleLogout() {
		const loginEndpoint = `${baseURL}/auth/logout`;
		try {
			const response = await fetch(loginEndpoint, {
				method: 'POST',
				credentials: 'include'
			});
			if (!response.ok) {
				throw new Error('Error getting Match Id');
			}
			goto('./');
		} catch (error) {
			if (error instanceof Error) {
				console.error(error.message);
			}
		}
	}

	//Get dog breeds
	async function getBreeds() {
		const loginEndpoint = `${baseURL}/dogs/breeds`;
		try {
			const response = await fetch(loginEndpoint, {
				credentials: 'include'
			});
			if (!response.ok) {
				throw new Error('Error getting Match Id');
			}
			const json = await response.json();
			breedsArray.push(...json);
		} catch (error) {
			if (error instanceof Error) {
				console.error(error.message);
			}
		}
	}

	function getDogSearchEndpoints() {
		let endpoint = `/dogs/search${getSearchSortEndpoints()}&ageMin=${ageMin}&ageMax=${ageMax}`;

		//Add endpoints for selected breeds with square bracket notation
		if (Array.isArray(selectedBreed)) {
			selectedBreed.forEach((breed, i) => {
				endpoint = endpoint + `&breeds[${i}]=${breed}`;
			});
		}
		return endpoint;
	}

	//handles logic for deciding what search sort endpoint to use
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

	//Search for dogs and get their Ids
	async function fetchDogIds() {
		const tempDogIdsArray = [];
		let endpoint = getDogSearchEndpoints();
		let nextExists;
		//pagination loop
		do {
			if (tempDogIdsArray.length >= 100) {
				break;
			}
			try {
				const response = await fetch(`${baseURL}${endpoint}`, {
					credentials: 'include'
				});
				if (!response.ok) {
					throw new Error('Error getting Match Id');
				}
				const json = await response.json();
				//check if there are more results
				if ('next' in json) {
					nextExists = true;
				} else {
					nextExists = false;
				}
				//update the endpoint to get next 25 results
				endpoint = json.next;
				tempDogIdsArray.push(...json.resultIds);
			} catch (error) {
				if (error instanceof Error) {
					console.error(error.message);
				}
			}
		} while (nextExists);

		dogIdsArray.push(...tempDogIdsArray);
		//Now that array of IDs is fetched, fetch dog objects
		fetchDogs(tempDogIdsArray);
	}

	//POST request to fetch dog objects
	async function fetchDogs(dogIdsArray: string[]) {
		try {
			const endpoint = `${baseURL}/dogs`;
			const response = await fetch(endpoint, {
				credentials: 'include',
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify(dogIdsArray)
			});
			if (!response.ok) {
				throw new Error('Error getting Match Id');
			}
			const json = await response.json();
			dogObjectArray = json;
		} catch (error) {
			if (error instanceof Error) {
				console.error(error.message);
			}
		}
	}

	//Add breed of the respective check box to the state array.
	//If the breed is already in the array, remove it
	function handleBreedCheckbox(breed: string) {
		if (selectedBreed.includes(breed)) {
			selectedBreed = selectedBreed.filter((item) => item !== breed);
		} else {
			selectedBreed.push(breed);
		}
	}

	//Checks the Id of each dog card with the favoritesArray to display whether it is favorited
	function toggleFavorites(id: string) {
		if (favoritesArray.includes(id)) {
			favoritesArray = favoritesArray.filter((item) => item !== id);
		} else {
			favoritesArray.push(id);
		}
	}

	//submits the favorited dogs to get the ID for the dog match
	async function getMatchId() {
		const endpoint = `${baseURL}/dogs/match`;
		try {
			const response = await fetch(endpoint, {
				credentials: 'include',
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify(favoritesArray)
			});
			if (!response.ok) {
				throw new Error('Error getting Match Id');
			}
			const json = await response.json();
			const matchId = json.match;
			//pass the Id that was fetched to then get the dog object
			fetchMatchObject([matchId]);
		} catch (error) {
			if (error instanceof Error) {
				console.error(error.message);
			}
			favoritesFlag = true;
		}
	}

	//used to get the city and state for the dog match
	async function getMatchLocationObject(matchZip: string) {
		const endpoint = `${baseURL}/locations`;
		try {
			const response = await fetch(endpoint, {
				credentials: 'include',
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify([matchZip])
			});
			if (!response.ok) {
				throw new Error('Error getting Match Id');
			}
			const json = await response.json();
			matchLocationObject = json[0];
		} catch (error) {
			if (error instanceof Error) {
				console.error(error.message);
			}
		}
	}

	//POST request to fetch dog objects
	async function fetchMatchObject(matchId: string[]) {
		const endpoint = `${baseURL}/dogs`;
		try {
			const response = await fetch(endpoint, {
				credentials: 'include',
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify(matchId)
			});
			if (!response.ok) {
				throw new Error(`Response status: ${response.status}`);
			}
			const json = await response.json();
			getMatchLocationObject(json[0].zip_code);
			favoritesFlag = false;
			matchObject = json[0];
		} catch (error) {
			if (error instanceof Error) {
				console.error(error.message);
			}
		}
	}
</script>

<div class="top-section">
	<section>
		<h1>Fetch Dog Matcher</h1>
		<div>Find the right dog for you!</div>
		<hr />
		<p class="instructions">
			1. Choose your favorite breeds, select your Age Range, and sort your search.
		</p>

		<div class="filters-container">
			<div>
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
			</div>
			<div>
				<div class="filter">
					Age Range:
					<div class="ages">
						<label>
							Min:
							<select name="Minimum Age" bind:value={ageMin}>
								{#each { length: 21 }, index}
									<option value={index}>{index}</option>
								{/each}
							</select>
						</label>

						<label>
							Max:
							<select name="Maximum Age" bind:value={ageMax}>
								{#each { length: 21 }, index}
									<option value={index}>{index}</option>
								{/each}
							</select>
						</label>
					</div>
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
			</div>
		</div>
		<p class="instructions">
			2. As you search, click the "Favorite" button for the dogs you love most.
		</p>
		<p class="instructions">
			3. Once you're finished, click the "Get Your Match!" button, and we'll pick the best dog for
			you.
		</p>
	</section>
</div>
<div class="bottom-section" bind:this={scrollToMatchRef}>
	<section>
		<!-- If a match has already been made, the button text changes to "Get New Match?" -->
		{#if !matchObject}
			<button
				type="button"
				onclick={() => {
					getMatchId();
					scrollToMatchRef.scrollIntoView({ behavior: 'smooth' });
				}}
				class="big-button narrow-button">Get Your Match!</button
			>
		{:else}
			<button
				type="button"
				onclick={() => {
					getMatchId();
					scrollToMatchRef.scrollIntoView({ behavior: 'smooth' });
				}}
				class="big-button narrow-button">Get New Match?</button
			>
		{/if}

		{#if favoritesFlag}
			<div class="warning">No match. Be sure you selected your favorites!</div>
		{/if}

		{#if matchObject}
			<div class="match-container">
				<div class="dog-card match-card">
					<h1 class="margin-bottom-small">You've been matched with {matchObject.name}!</h1>
					<img src={matchObject.img} alt="Dog match" />
					<div>Name: {matchObject.name}</div>
					<div>Age: {matchObject.age}</div>
					<div>Breed: {matchObject.breed}</div>
					<div class="margin-bottom-small">Zipcode: {matchObject.zip_code}</div>
					<div>
						<!-- Give different match messages based on the dog's age -->
						{#if matchLocationObject}
							Congratulations on your match. {matchObject.name}
							{#if matchObject.age <= 2}
								is a nice lil' pup from {matchLocationObject.city}, {matchLocationObject.state} with
								a full life to live.
							{:else if matchObject.age > 2 && matchObject.age <= 10}
								is an adolescent doggy from {matchLocationObject.city}, {matchLocationObject.state} with
								lots of energy.
							{:else}
								from {matchLocationObject.city}, {matchLocationObject.state} is happy to have a cozy
								place to spend retirement.
							{/if}
						{/if}
					</div>
				</div>
			</div>
		{/if}
		<div class="page-numbers">
			{#each { length: numberOfPages }, index}
				<button
					onclick={() => {
						currentPage = index + 1;
					}}
					class={index + 1 === currentPage ? 'page-number active' : 'page-number'}
					>{index + 1}</button
				>
			{/each}
		</div>
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
						scrollToMatchRef.scrollIntoView({ behavior: 'smooth' });
					}}
					class={index + 1 === currentPage ? 'page-number active' : 'page-number'}
					>{index + 1}</button
				>
			{/each}
		</div>
		<button type="button" onclick={handleLogout} class="big-button narrow-button">Logout</button>
	</section>
</div>
