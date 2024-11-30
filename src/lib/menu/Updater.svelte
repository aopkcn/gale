<script context="module" lang="ts">
	import { check, type Update } from '@tauri-apps/plugin-updater';
	import { writable } from 'svelte/store';

	export let nextUpdate = writable<Update | null>(null);
	export let isChecking = writable(false);

	export async function refreshUpdate() {
		isChecking.set(true);
		let update = await check();
		isChecking.set(false);

		if (update === null || !update.available) return;
		nextUpdate.set(update);
	}
</script>

<script lang="ts">
	import BigButton from '$lib/components/BigButton.svelte';
	import ConfirmPopup from '$lib/components/ConfirmPopup.svelte';
	import { pushError } from '$lib/invoke';
	import Icon from '@iconify/svelte';
	import { message } from '@tauri-apps/plugin-dialog';
	import { platform } from '@tauri-apps/plugin-os';
	import { relaunch } from '@tauri-apps/plugin-process';
	import { Button, Dialog } from 'bits-ui';
	import { onMount } from 'svelte';

	let popupOpen = false;
	let loading = false;

	onMount(() => {
		refreshUpdate();
	});

	async function installUpdate() {
		if ($nextUpdate === null) return;

		try {
			await $nextUpdate.downloadAndInstall();
		} catch (e) {
			let message: string;
			if (typeof e === 'string') {
				message = e;
			} else if (e instanceof Error) {
				message = e.message;
			} else {
				message = 'Unknown error';
			}

			pushError(
				{
					name: 'Failed to update Gale',
					message
				},
				true
			);
		}

		$nextUpdate = null;
	}

	async function update() {
		popupOpen = false;
		loading = true;
		await installUpdate();
		loading = false;

		if (platform() !== 'windows') {
			// on other platforms installUpdate() relaunches the app itself
			await message('Gale will now restart in order to apply the update.');
			await relaunch();
		}
	}
</script>

{#if $nextUpdate != null}
	<Button.Root
		class="my-auto ml-auto mr-1.5 flex items-center rounded-md bg-accent-600 
            px-2.5 py-1 font-semibold text-slate-100 enabled:hover:bg-accent-500"
		disabled={loading}
		on:click={() => (popupOpen = true)}
	>
		{#if loading}
			<Icon icon="mdi:loading" class="mr-1 animate-spin" />
		{:else}
			<Icon icon="mdi:arrow-up-circle" class="mr-1" />
		{/if}
		<span class="text-sm">{loading ? '正在下载更新...' : '有更新'}</span>
	</Button.Root>
{/if}

<ConfirmPopup title="应用程序更新可用" bind:open={popupOpen}>
	<Dialog.Description class="text-slate-300">
		<p>
			{#if nextUpdate}
				版本 {$nextUpdate?.version} 的 Gale 已经可用 - 当前版本是 {$nextUpdate?.currentVersion}。
			{:else}
				Gale 有一个可用的更新。
			{/if}

			更新将在后台下载，然后应用程序将重新启动以进行更新。
		</p>
		<p class="mt-1">您想要安装它吗？</p>
	</Dialog.Description>

	<svelte:fragment slot="buttons">
		<BigButton color="accent" fontWeight="semibold" on:click={update}>安装</BigButton>
	</svelte:fragment>
</ConfirmPopup>
