<script>
	import { onMount } from "svelte";
	import { ConnectWallet } from "@proton/web-sdk";
	// Constants
	const appIdentifier = "taskly";
	let link, session;

	async function createLink({ restoreSession }) {
		const result = await ConnectWallet({
			linkOptions: {
				endpoints: ["https://proton.greymass.com"],
				restoreSession,
			},
			transportOptions: {
				requestAccount: "myprotonacc", // Your proton account
				requestStatus: true,
			},
			selectorOptions: {
				appName: "Taskly",
				appLogo:
					"https://taskly.protonchain.com/static/media/taskly-logo.ad0bfb0f.svg",
				customStyleOptions: {
					modalBackgroundColor: "#F4F7FA",
					logoBackgroundColor: "white",
					isLogoRound: true,
					optionBackgroundColor: "white",
					optionFontColor: "black",
					primaryFontColor: "black",
					secondaryFontColor: "#6B727F",
					linkColor: "#752EEB",
				},
			},
		});
		link = result.link;
		session = result.session;
	}

	async function login() {
		// Create link
		await createLink({ restoreSession: false });
		console.log("User authorization:", session.auth); // { actor: 'fred', permission: 'active }
	}

	async function transfer() {
		// Send Transaction
		const result = await session.transact({
			transaction: {
				actions: [
					{
						// Token contract for FOOBAR
						account: "xtokens",
						// Action name
						name: "transfer",
						// Action parameters
						data: {
							from: session.auth.actor,
							to: "freeosclaimb",
							quantity: "0.100000 FOOBAR",
							memo: "Tip!",
						},
						authorization: [session.auth],
					},
				],
			},
			broadcast: true,
		});
		console.log("Transaction ID", result.processed.id);
	}

	async function logout() {
		await link.removeSession(appIdentifier, session.auth);
		session = undefined;
	}

	async function reconnect() {
		try {
			await createLink({ restoreSession: true });
		} catch (e) {
			console.warn(e);
		}
	}

	onMount(() => {
		reconnect();
	});
</script>

<main>
	{#if session}
		<h1>Account: {session.auth.actor}</h1>
		<button class="app-button" on:click={transfer}>Transfer</button>
		<button class="app-button" on:click={logout}>Logout</button>
	{:else}
		<button class="app-button" on:click={login}>Login</button>
	{/if}
</main>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 2em;
		font-weight: 100;
	}

	.app-button {
		font-weight: 600;
		font-size: 1.125rem;
		padding: 10px 25px;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}
</style>
