<script>
	let { id, img, name, age, zip_code, breed } = $props();
</script>

<div class="dog-card">
	<img src={img} alt="A dog from search results" />
	<div>Name: {name}</div>
	<div>ID: {id}</div>
	<div>Age: {age}</div>
	<div>Breed: {breed}</div>
	<div>Zipcode {zip_code}</div>
</div>
