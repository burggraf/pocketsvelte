<script context="module" lang="ts">
	import { browser, dev } from '$app/env';
    import PocketBase from 'pocketbase'
    const client = new PocketBase('http://localhost:8090');
    const redirectUrl = 'http://localhost:8100/redirect';

	// we don't need any JS on this page, though we'll load
	// it in dev so that we get hot module replacement...
	export const hydrate = dev;

	// ...but if the client-side router is already loaded
	// (i.e. we came here from elsewhere in the app), use it
	export const router = browser;

	// since there's no dynamic data here, we can prerender
	// it so that it gets served as a static asset in prod
	export const prerender = true;
    let email = '';
    let password = '';
    let providers: any[] = [];
    const getProviders = async () => {
        return (await client.Users.listAuthMethods())
        .authProviders;
    }

    const login = async (email: string, password: string) => {
		console.log('client', client)
		console.log('email', email)
		console.log('password', password)

		try {
			const authData = await client.Users.authViaEmail(email, password)
			console.log('authData', authData)
		} catch (err) {
			console.log('error', err)
		}

    };
	const loginWithProvider = async (provider: any) => {
		console.log('loginWithProvider', provider)
        localStorage.setItem("provider", JSON.stringify(provider));
        console.log('provider.authUrl', provider.authUrl);
        console.log('redirectUrl', redirectUrl);
        window.location.href = provider.authUrl + redirectUrl;
	}
	const logout = async () => {
		console.log('client', client)
		client.AuthStore.clear()
		//client.AuthStore.
	}

</script>

<!-- svelte-ignore module-script-reactive-declaration -->
<!-- svelte-ignore module-script-reactive-declaration -->
<svelte:head>
	<title>Login</title>
	<meta name="description" content="Login to the app" />
</svelte:head>

<div class="content">
	<h1>Login</h1>
    Email: <input bind:value={email} type="text" name="email" placeholder="email" />
    <br/>
    Password: <input bind:value={password} type="password" name="password" placeholder="password" />
    <br/>
    <button on:click={() => {
        login(email, password);
    }}>Login</button>

    <button on:click={() => {
        logout();
    }}>Logout</button>

    {#await getProviders()}
        <p>Loading</p>
    {:then providers}
        {#each providers as provider}
            <br/>
            <button on:click={() => {
                loginWithProvider(provider);
            }}>Login with {
                provider.name.charAt(0).toUpperCase() + provider.name.slice(1)
            }
            </button>
    	{/each}
    {/await}

</div>

<style>
	.content {
		width: 100%;
		max-width: var(--column-width);
		margin: var(--column-margin-top) auto 0 auto;
	}
</style>
