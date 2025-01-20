<script lang="ts">
	import '../app.css';
	import { goto } from '$app/navigation';

	const baseURL = 'https://frontend-take-home-service.fetch.com';
	let loginName = $state('John Doe');
	let loginEmail = $state('john@doemail.com');
	let validEmail = $state(false);
	let loginAttempt = $state(false);

	//check that the email is a valid email format
	function validateEmail(email: string) {
		let re = /\S+@\S+\.\S+/;
		let valid = re.test(email);
		if (valid) {
			validEmail = true;
		}
		return valid;
	}

	//Login function
	async function handleLogin() {
		loginAttempt = true;
		try {
			if (validateEmail(loginEmail)) {
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
				console.log(response);
				if (!response.ok) {
					throw new Error(`Response status: ${response.status}. Error logging in.`);
				}
				if (response.ok) {
					goto('/app');
				}
			}
		} catch (error) {
			if (error instanceof Error) {
				console.error(error.message);
			}
		}
	}
</script>

<section class="login-section">
	<h1>Fetch Dog Matcher</h1>
	<div>Find the right dog for you!</div>
	<div class="login-inputs">
		<label
			>Name:
			<input bind:value={loginName} type="text" required />
		</label>
		<label>
			Email:
			<input bind:value={loginEmail} type="email" required />
		</label>
	</div>
	<!-- conditionally render valid email prompt -->
	{#if !validEmail && loginAttempt}
		<div>Please enter a valid email.</div>
	{/if}

	<button type="button" onclick={handleLogin} class="big-button">Login</button>
</section>
