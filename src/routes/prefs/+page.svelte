<script lang="ts">
	import PathPref from '$lib/prefs/PathPref.svelte';
	import LaunchModePref from '$lib/prefs/LaunchModePref.svelte';
	import ZoomLevelPref from '$lib/prefs/ZoomFactorPref.svelte';
	import TogglePref from '$lib/prefs/TogglePref.svelte';
	import ApiKeyPref from '$lib/prefs/ApiKeyPref.svelte';
	import ApiKeyPopup from '$lib/prefs/ApiKeyPopup.svelte';

	import { activeGame } from '$lib/stores';
	import { Separator } from 'bits-ui';
	import { type Prefs, type GamePrefs, Platform } from '$lib/models';
	import { onMount } from 'svelte';
	import { invokeCommand } from '$lib/invoke';
	import CustomArgsPref from '$lib/prefs/CustomArgsPref.svelte';
	import AccentColorPref from '$lib/prefs/AccentColorPref.svelte';
	import LargePrefsHeading from '$lib/prefs/LargePrefsHeading.svelte';
	import SmallPrefsHeading from '$lib/prefs/SmallPrefsHeading.svelte';
	import PlatformPref from '$lib/prefs/PlatformPref.svelte';
	import { platform } from '@tauri-apps/plugin-os';

	let prefs: Prefs | null = null;
	let gamePrefs: GamePrefs | null = null;

	$: gameSlug = $activeGame?.slug ?? '';
	$: gamePrefs = prefs?.gamePrefs.get(gameSlug) ?? {
		launchMode: { type: 'launcher' },
		dirOverride: null,
		customArgs: null,
		platform: null
	};

	$: platforms = $activeGame?.platforms ?? [];
	$: needsDirectory = !platforms.some(
		(p) =>
			p === Platform.Steam ||
			(platform() === 'windows' && (p === Platform.EpicGames || p === Platform.XboxStore))
	);

	onMount(async () => {
		let newPrefs = await invokeCommand<Prefs>('get_prefs');
		newPrefs.gamePrefs = new Map(Object.entries(newPrefs.gamePrefs));
		prefs = newPrefs;
	});

	function set<T>(update: (value: T, prefs: Prefs) => void) {
		return async (value: T) => {
			if (prefs === null) return;

			update(value, prefs);
			prefs.gamePrefs.set(gameSlug, gamePrefs!);
			await invokeCommand('set_prefs', { value: prefs });
		};
	}
</script>

<div class="flex w-full flex-col gap-1 overflow-y-auto px-6 pb-6 pt-2">
	{#if prefs !== null && gamePrefs !== null}
		<LargePrefsHeading>全局设置</LargePrefsHeading>

		<SmallPrefsHeading>位置</SmallPrefsHeading>

		<PathPref
			label="Steam 可执行文件"
			type="file"
			value={prefs.steamExePath ?? null}
			set={set((value, prefs) => (prefs.steamExePath = value))}
		>
		Steam 可执行文件（steam.exe）的路径，用于启动 Steam。
		</PathPref>

		<PathPref
			label="Steam 库"
			type="dir"
			value={prefs.steamLibraryDir ?? null}
			set={set((value, prefs) => (prefs.steamLibraryDir = value))}
		>
		默认 Steam 游戏库的路径，用于查找 Steam 游戏的位置。
		</PathPref>

		<PathPref
			label="数据目录"
			type="dir"
			value={prefs.dataDir}
			set={set((value, prefs) => (prefs.dataDir = value))}
		>
		存储插件和配置文件的目录。更改此目录将移动现有数据。
		</PathPref>

		<SmallPrefsHeading>外观</SmallPrefsHeading>

		<AccentColorPref />

		<ZoomLevelPref
			value={prefs.zoomFactor}
			set={set((value, prefs) => (prefs.zoomFactor = value))}
		/>

		<SmallPrefsHeading>其他</SmallPrefsHeading>

		<ApiKeyPref />

		<TogglePref
			label="自动获取插件"
			value={prefs.fetchModsAutomatically}
			set={set((value, prefs) => (prefs.fetchModsAutomatically = value))}
		>
		是否每 15 分钟自动获取插件。这将确保插件列表保持相对最新，但可以禁用以节省带宽。
		<br />
		要手动触发获取，请前往 <b>文件 &gt; 获取插件</b>。
		</TogglePref>

		<LargePrefsHeading>
			{$activeGame?.name} 设置
		</LargePrefsHeading>

		<SmallPrefsHeading>位置</SmallPrefsHeading>

		{#if platforms.length > 0}
			<PlatformPref value={gamePrefs.platform} set={set((value) => (gamePrefs.platform = value))} />
		{/if}

		<PathPref
			label={needsDirectory ? '游戏目录' : '游戏路径'}
			type="dir"
			canClear={true}
			value={gamePrefs.dirOverride}
			set={set((value) => (gamePrefs.dirOverride = value))}
		>
		覆盖 {$activeGame?.name} 游戏目录的路径。
		{#if !needsDirectory}
			如果未设置，Gale 将尝试通过指定的 Steam 库来查找它。
		{/if}
		</PathPref>

		<SmallPrefsHeading>启动</SmallPrefsHeading>

		<LaunchModePref
			value={gamePrefs.launchMode}
			set={set((value) => (gamePrefs.launchMode = value))}
		/>

		<CustomArgsPref
			value={gamePrefs.customArgs}
			set={set((value) => (gamePrefs.customArgs = value))}
		/>
	{/if}
</div>

<ApiKeyPopup />
