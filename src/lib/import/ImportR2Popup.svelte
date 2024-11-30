<script lang="ts">
	import BigButton from '$lib/components/BigButton.svelte';
	import Popup from '$lib/components/Popup.svelte';
	import { invokeCommand } from '$lib/invoke';
	import ImportR2Flow from './ImportR2Flow.svelte';
	import type { R2ImportData } from '$lib/models';
	import { onMount } from 'svelte';

	export let open: boolean;

	let loading = false;
	let importFlow: ImportR2Flow;
	let importData: R2ImportData;
	let importFrom: 'r2modman' | 'thunderstore';

	$: if (open) {
		load();
	}

	async function load() {
		importData = await invokeCommand<R2ImportData>('get_r2modman_info');

		if (importData.r2modman) {
			importData.r2modman.include = importData.r2modman.profiles.map(() => true);
			importFrom = 'r2modman';
		}

		if (importData.thunderstore) {
			importData.thunderstore.include = importData.thunderstore.profiles.map(() => true);
			importFrom = 'thunderstore';
		}
	}

	async function doImport() {
		await importFlow.doImport();
		open = false;
	}
</script>

<Popup bind:open title="从其他管理器导入配置文件" canClose={!loading}>
	<div class="text-slate-300">
		<p>
			这将从 r2modman 或 Thunderstore Mod Manager 导入 <b>当前游戏</b> 的配置文件。
		</p>

		<p class="mt-2">
			该过程可能需要几分钟，具体取决于要导入的模组数量。
			<b>同名的现有配置文件将被覆盖！</b>
		</p>

		<p class="mt-2">在导入过程中请勿关闭 Gale。.</p>
	</div>

	<ImportR2Flow bind:this={importFlow} bind:importData bind:importFrom bind:loading />

	<div class="mr-0.5 mt-3 flex w-full justify-end gap-2">
		{#if importData?.r2modman || importData?.thunderstore}
			<BigButton color="slate" on:click={() => (open = false)}>取消</BigButton>
			<BigButton color="accent" on:click={doImport}>导入</BigButton>
		{/if}
	</div>
</Popup>
