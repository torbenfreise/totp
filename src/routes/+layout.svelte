<script lang="ts">
	import './+layout.css';
	import favicon from '$lib/assets/favicon.svg';

	let { children } = $props();
</script>

<svelte:head
	><link rel="icon" href={favicon} />
	<title>TOTP Generator (RFC 6238) – Generate 2FA Authentication Codes</title>
	<link rel="canonical" href="https://totp.torbenfreise.dev" />
	<meta
		name="description"
		content="Generate RFC 6238–compliant time-based one-time passwords (TOTP) for 2FA authentication."
	/>
</svelte:head>
{@render children()}
