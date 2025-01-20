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
	const breedsArray = $state([]);
	let selectedBreed = $state([]);
	let searchSort = $state('breed-asc');
	let ageMin = $state(0);
	let ageMax = $state(20);
	let favoritesArray = $state([]);
	let matchObject = $state();
	let matchLocationObject = $state('');
	// let cityName = $state('Chicago');
	// let zipCodesArray = [];
	//reference to "Get Your Match!" button to be scrolled to
	let scrollToMatchRef;

	//Automatically update search results upon changing any sorting/filtering/breed selection
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
			selectedBreed.forEach((breed, i) => {
				endpoint = endpoint + `&breeds[${i}]=${breed}`;
			});
		}

		// getCityZipCodes(cityName);
		// console.log(zipCodesArray);
		// //adds endpoints for zipcode filter
		// if (zipCodesArray.length > 0) {
		// 	zipCodesArray.forEach((zipCode, i) => {
		// 		endpoint = endpoint + `&zipCodes[${i}]=${zipCode}`;
		// 	});
		// }
		// // endpoint = endpoint + '&zipCodes[0]=60634';
		// console.log(endpoint);
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

	// async function getCityZipCodes(passedCity) {
	// 	const endpoint = `${baseURL}/locations/search`;
	// 	const response = await fetch(endpoint, {
	// 		credentials: 'include',
	// 		method: 'POST',
	// 		headers: { 'Content-Type': 'application/json' },
	// 		body: JSON.stringify({
	// 			city: passedCity,
	// 			size: 100
	// 		})
	// 	});
	// 	const json = await response.json();
	// 	const objectArray = json.results;
	// 	const tempZipCodesArray = [];
	// 	//extract the zip codes from each object returned in the array, and push them to a separate array
	// 	objectArray.forEach((obj) => {
	// 		tempZipCodesArray.push(obj.zip_code);
	// 	});
	// 	console.log(tempZipCodesArray);
	// 	console.log(objectArray);
	// 	// zipCodesArray.push(...tempZipCodesArray);
	// 	zipCodesArray = tempZipCodesArray;
	// }

	//Search for dogs
	async function fetchDogIds() {
		const tempDogIdsArray = [];
		let endpoint = getDogSearchEndpoints();
		console.log(endpoint);
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

	//Add breed of the respective check box to the state array
	//If the breed is already in the array, remove it
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
		console.log(matchId);

		fetchMatchObject([matchId]);
	}

	//
	async function getMatchLocationObject(matchZip) {
		console.log(matchZip);
		const endpoint = `${baseURL}/locations`;
		const response = await fetch(endpoint, {
			credentials: 'include',
			method: 'POST',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify([matchZip])
		});
		const json = await response.json();
		console.log(json[0].city);
		matchLocationObject = json[0];
		console.log(matchLocationObject.city);
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
		console.log(response);

		const json = await response.json();
		getMatchLocationObject(json[0].zip_code);
		matchObject = json[0];
	}
</script>

<div class="top-section">
	<section>
		<h1>Fetch Dog Matcher</h1>
		<div>Find the right dog for you!</div>
		<p class="instructions">
			Choose your favorite breeds and filter your search by age and location.
		</p>
		<hr />
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
		</div>

		<!-- <div class="filter">
                City:
                <label>
                    <input type="text" bind:value={cityName} />
                </label>
            </div> -->
		<p class="instructions">
			As you search, click the "Add to favorites" button for the dogs you love most. Once you're
			finished, click the "Get Your Match!" button, and we'll pick the best dog for you.
		</p>
	</section>
</div>
<div class="bottom-section">
	<section>
		<button
			type="button"
			bind:this={scrollToMatchRef}
			onclick={() => {
				getMatchId();
				scrollToMatchRef.scrollIntoView({ alignToTop: 'true', behavior: 'smooth' });
			}}
			class="big-button">Get Your Match!</button
		>
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
		{#if matchObject}
			<div class="match-container">
				<div class="dog-card match-card">
					<h1>You've been matched with {matchObject.name}!</h1>
					<img src={matchObject.img} alt="Dog match" />
					<div>Name: {matchObject.name}</div>
					<div>Age: {matchObject.age}</div>
					<div>Breed: {matchObject.breed}</div>
					<div>Zipcode: {matchObject.zip_code}</div>
					<div>
						Congratulations on your match. {matchObject.name}
						{#if matchObject.age <= 2}
							is a nice lil' pup from {matchLocationObject.city}, {matchLocationObject.state} with a
							full life to live.
						{:else if matchObject.age > 2 && matchObject.age <= 10}
							is an adolescent doggy from {matchLocationObject.city}, {matchLocationObject.state} with
							lots of energy.
						{:else}
							from {matchLocationObject.city}, {matchLocationObject.state} is happy to have a cozy place
							to spend retirement.
						{/if}
					</div>
				</div>
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
					class={index + 1 === currentPage ? 'page-number active' : 'page-number'}
					>{index + 1}</button
				>
			{/each}
		</div>
		<button type="button" onclick={handleLogout} class="big-button">Logout</button>
	</section>
</div>
