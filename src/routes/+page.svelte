<script lang="ts">
	import { generate, type HashAlgorithm } from 'otplib';
	import { type Digits } from '@otplib/core';
	import { onDestroy, onMount } from 'svelte';
	import { Input, Progressbar, Select } from 'flowbite-svelte';
	import { Clipboard } from 'flowbite-svelte';
	import { CheckOutline, ClipboardCleanSolid } from 'flowbite-svelte-icons';
	import { goto } from '$app/navigation';
	import { SvelteURLSearchParams } from 'svelte/reactivity';
	import { resolve } from '$app/paths';

	let secret = $state('ORXXEYTFNYXGM4TFNFZWKQDHNVQWS3BOMNXW2SCFJZHEORKDJBAUYTCFJZDUKMBQGQ======');

	onMount(() => {
		const params = new SvelteURLSearchParams(window.location.search);
		const fromUrl = params.get('secret');
		if (fromUrl) secret = fromUrl;
	});

	let algorithm = $state<HashAlgorithm>('sha1');

	const minPeriod = 1;
	const maxPeriod = 100;
	let period = $state(30);

	const minDigits = 0;
	const maxDigits = 12;
	let digits = $state<Digits>(6);

	let now = $state(Date.now());
	const timestep = $derived(Math.floor(now / 1000 / period));

	// 👇 single derived config
	const totpOptions = $derived({
		secret,
		algorithm,
		digits,
		period
	});

	const totpCode = $derived(timestep && generate(totpOptions));

	// Seconds remaining in current timestep
	const remaining = $derived(period - (Math.floor(now / 1000) % period));
	let success = $state(false);

	const interval = setInterval(() => {
		now = Date.now();
	}, 1000);
	const clamp = (v: number, min: number, max: number) => Math.min(max, Math.max(min, v));

	function updateUrl(value: string) {
		const searchParams = new SvelteURLSearchParams();
		const url = resolve('/');
		if (value) searchParams.set('secret', value);

		// eslint-disable-next-line svelte/no-navigation-without-resolve
		goto(`${url}?${searchParams.toString()}`, {
			replaceState: true,
			noScroll: true,
			keepFocus: true
		});
	}

	onDestroy(() => clearInterval(interval));
</script>

<div class="flex min-h-screen items-center justify-center bg-gray-50">
	<div class="w-full max-w-6xl px-6">
		<div class="grid grid-cols-1 items-start gap-12 lg:grid-cols-2">
			<!-- Left: text -->
			<div class="space-y-4 text-gray-700">
				<h1 class="text-3xl font-semibold text-gray-800">Time-based One-Time Passwords</h1>

				<p>
					This tool generates TOTP codes according to <a
						href="https://www.rfc-editor.org/rfc/rfc6238"
						class="text-link-700 hover:text-link-800 dark:text-link-300 dark:hover:text-link-100 font-medium underline"
					>
						RFC 6238</a
					>.
				</p>

				<p>You can choose the hash algorithm, token length, and time step.</p>

				<p>Secrets must be provided in Base-32 encoding.</p>
			</div>

			<!-- Right: generator card -->
			<!-- Right: generator card -->
			<div class="space-y-6 rounded-xl bg-white p-6 shadow">
				<div class="w-full max-w-xl space-y-6 rounded-xl bg-white p-6 shadow">
					<h1 class="text-center text-2xl font-semibold text-gray-800">TOTP Generator</h1>

					<!-- Algorithm -->
					<div class="space-y-1">
						<label for="Algorithm" class="text-sm font-medium text-gray-600">Hash algorithm</label>
						<Select id="Algorithm" bind:value={algorithm} class="w-full">
							<option value="sha1">SHA-1</option>
							<option value="sha256">SHA-256</option>
							<option value="sha512">SHA-512</option>
						</Select>
					</div>

					<!-- Period -->
					<div class="space-y-1">
						<label for="Period" class="text-sm font-medium text-gray-600">Period (seconds)</label>
						<Input
							id="Period"
							type="number"
							bind:value={period}
							min={minPeriod}
							max={maxPeriod}
							oninput={() => (period = clamp(period, minPeriod, maxPeriod) as Digits)}
							class="w-full"
						/>
					</div>

					<!-- Digits -->
					<div class="space-y-1">
						<label for="Digits" class="text-sm font-medium text-gray-600">Code length</label>
						<Input
							id="Digits"
							type="number"
							bind:value={digits}
							min={minDigits}
							max={maxDigits}
							oninput={() => (digits = clamp(digits, minDigits, maxDigits) as Digits)}
							class="w-full"
						/>
					</div>

					<!-- Secret -->
					<div class="space-y-1">
						<label for="secret" class="text-sm font-medium text-gray-600">Base32 Secret</label>
						<Input
							id="secret"
							type="text"
							bind:value={secret}
							oninput={() => updateUrl(secret)}
							onchange={() => updateUrl(secret)}
							class="w-full"
						/>
					</div>

					<!-- Result -->
					<div class="space-y-3 border-t pt-4 text-center">
						<h2 class="text-lg font-medium text-gray-700">TOTP Code</h2>

						{#await totpCode then code}
							<div class="flex items-center justify-center gap-3 font-mono text-3xl">
								<span>{code}</span>

								<Clipboard
									size="xs"
									color={success ? 'alternative' : 'light'}
									bind:success
									value={code.toString()}
								>
									{#if success}
										<CheckOutline class="h-4 w-4" />
									{:else}
										<ClipboardCleanSolid class="h-4 w-4" />
									{/if}
								</Clipboard>
							</div>
						{/await}

						<p class="text-sm text-gray-500">
							Expires in {remaining}s
						</p>

						<Progressbar progress={(remaining / period) * 100} />
					</div>
				</div>
			</div>
		</div>
	</div>
</div>
