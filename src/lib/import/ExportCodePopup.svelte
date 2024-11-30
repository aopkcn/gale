<script lang="ts">
	import Popup from '$lib/components/Popup.svelte';
	import { invokeCommand } from '$lib/invoke';
	import { activeProfile, refreshProfiles } from '$lib/stores';
	import Icon from '@iconify/svelte';
	import { writeText } from '@tauri-apps/plugin-clipboard-manager';
	import { Dialog } from 'bits-ui';

	let isOpen = false;

	let codePromise: Promise<string>;

	export async function open() {
		codePromise = invokeCommand('export_code');
		isOpen = true;

		try {
			let code = await codePromise;
			await writeText(code);
		} catch (e) {
			isOpen = false;
		}
	}
</script>

<Popup title="导出为代码" bind:open={isOpen}>
	<Dialog.Description class="flex-center mb-2 flex text-slate-400">
		{#await codePromise}
			<Icon icon="mdi:loading" class="mr-2 animate-spin text-lg" />
			正在导出 {$activeProfile?.name} 作为代码...
		{:then}
			导出完成！代码已经复制到您的剪贴板
		{/await}
	</Dialog.Description>

	{#await codePromise then code}
		<code class="rounded bg-slate-900 px-3 py-1 text-lg text-slate-400">
			{code}
		</code>
	{/await}
</Popup>
