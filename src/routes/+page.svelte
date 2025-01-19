<script>
	import '../app.css';
	import { goto } from '$app/navigation';

	const baseURL = 'https://frontend-take-home-service.fetch.com';
	let loginName = 'Noah';
	let loginEmail = 'noah.mailloux1@gmail.com';

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
		if (response.ok) {
			goto('/app');
		}
	}
</script>

<label>
	<input bind:value={loginName} type="text" required />
</label>
<label>
	<input bind:value={loginEmail} type="email" required />
</label>
<button type="button" onclick={handleLogin}>Login</button>
