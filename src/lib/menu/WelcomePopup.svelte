<script lang="ts">
	import GameSelection from '$lib/components/GameSelection.svelte';
	import Popup from '$lib/components/Popup.svelte';
	import BigButton from '$lib/components/BigButton.svelte';
	import PathPref from '$lib/prefs/PathPref.svelte';

	import type { Prefs, R2ImportData } from '$lib/models';

	import { invokeCommand } from '$lib/invoke';
	import { onMount } from 'svelte';
	import ImportR2Flow from '$lib/import/ImportR2Flow.svelte';
	import Icon from '@iconify/svelte';

	export let open = false;

	let stage: 'gameSelect' | 'importProfiles' | 'settings' | 'end' = 'gameSelect';

	let importFrom: 'r2modman' | 'thunderstore' = 'r2modman';
	let importData: R2ImportData = {
		r2modman: null,
		thunderstore: null
	};

	let importFlow: ImportR2Flow;
	let prefs: Prefs | null = null;

	$: importText =
		importData.r2modman && importData.thunderstore
			? 'r2modman or Thunderstore Mod Manager'
			: importData.r2modman
				? 'r2modman'
				: 'Thunderstore Mod Manager';

	onMount(async () => {
		if (await invokeCommand<boolean>('is_first_run')) {
			open = true;
			prefs = await invokeCommand('get_prefs');
		}
	});

	async function onSelectGame() {
		let result = await invokeCommand<R2ImportData>('get_r2modman_info');

		if (!result.r2modman && !result.thunderstore) {
			stage = 'settings';
			return;
		}

		importData = result;

		if (result.r2modman) {
			result.r2modman.include = result.r2modman.profiles.map(() => true);
			importFrom = 'r2modman';
		}

		if (result.thunderstore) {
			result.thunderstore.include = result.thunderstore.profiles.map(() => true);
			importFrom = 'thunderstore';
		}

		stage = 'importProfiles';
	}

	async function importProfiles() {
		await importFlow.doImport();
		stage = 'settings';
	}

	function set<T>(update: (value: T, prefs: Prefs) => void) {
		return async (value: T) => {
			if (prefs === null) return;

			update(value, prefs);
			await invokeCommand('set_prefs', { value: prefs });
		};
	}
</script>

<Popup title="欢迎来到 Gale!" canClose={stage === 'end'} bind:open>
	<div class="text-slate-300">
		{#if stage === 'gameSelect'}
			首先，选择一个游戏进行修改：
			<GameSelection onSelect={onSelectGame} />
		{:else if stage === 'importProfiles' && importData}
			<p>
				您可以选择自动将配置文件从 {importText} 转移到 Gale。
			</p>

			<p class="mt-1">
				这个过程可能需要几分钟，具体取决于需要导入的修改和配置文件数量。
			</p>

			<p class="mt-1">
				您也可以稍后通过进入 <b>导入 &gt; ...从 r2modman 导入</b> 来导入配置文件。
			</p>

			<ImportR2Flow bind:importData bind:importFrom bind:this={importFlow} />

			<div class="mt-2 flex gap-1.5">
				<BigButton color="slate" class="mr-auto" on:click={() => (stage = 'gameSelect')}
					>返回</BigButton
				>
				<BigButton color="slate" on:click={() => (stage = 'settings')}>跳过</BigButton>
				<BigButton color="accent" on:click={importProfiles}>导入</BigButton>
			</div>
		{:else if stage === 'settings'}
			<p>
				让我们确保您的设置是正确的。
				<br />
				您随时可以通过进入 <Icon icon="mdi:settings" class="mb-1 inline" />
				<b>设置</b> 来编辑这些设置。
			</p>

			<div class="mt-3 flex flex-col gap-1">
				{#if prefs !== null}
					<PathPref
						label="Steam 执行文件"
						type="file"
						value={prefs.steamExePath}
						set={set((value, prefs) => (prefs.steamExePath = value))}
					>
						Steam 执行文件的路径。
					</PathPref>

					<PathPref
						label="Steam 库"
						type="dir"
						value={prefs.steamLibraryDir}
						set={set((value, prefs) => (prefs.steamLibraryDir = value))}
					>
						Steam 游戏库的路径。此路径应 <b>包含</b> 'steamapps' 目录。
					</PathPref>

					<PathPref
						label="Gale 数据目录"
						type="dir"
						value={prefs.dataDir}
						set={set((value, prefs) => (prefs.dataDir = value))}
					>
					 存储修改和配置文件的目录。
					</PathPref>
				{/if}
			</div>

			<div class="mt-3 flex justify-between">
				<BigButton
					color="slate"
					on:click={() =>
						(stage =
							importData.r2modman || importData.thunderstore ? 'importProfiles' : 'gameSelect')}>返回</BigButton>
				<BigButton color="accent" on:click={() => (stage = 'end')}>下一步</BigButton>
			</div>
		{:else if stage === 'end'}
			<p>就是这样，您已经准备好开始修改了！</p>

			<p class="mt-1">
				如果您有任何问题或需要帮助，请随时在 <a
					href="https://discord.gg/sfuWXRfeTt"
					target="_blank"
					class="text-accent-400 hover:underline">Discord 服务器</a>
				> 提问。
			</p>
		{/if}
	</div>
</Popup>

