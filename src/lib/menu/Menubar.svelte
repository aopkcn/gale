<script lang="ts">
	import { onMount } from 'svelte';
	import Icon from '@iconify/svelte';
	import { Button, Menubar } from 'bits-ui';

	import MenubarItem from '$lib/menu/MenubarItem.svelte';

	import InputField from '$lib/components/InputField.svelte';
	import BigButton from '$lib/components/BigButton.svelte';
	import Popup from '$lib/components/Popup.svelte';

	import ImportR2Popup from '$lib/import/ImportR2Popup.svelte';
	import ExportCodePopup from '$lib/import/ExportCodePopup.svelte';
	import ImportProfilePopup from '$lib/import/ImportProfilePopup.svelte';

	import AboutPopup from './AboutPopup.svelte';
	import MenubarMenu from './MenubarMenu.svelte';
	import NewProfilePopup from './NewProfilePopup.svelte';
	import MenubarSeparator from './MenubarSeparator.svelte';

	import { capitalize } from '$lib/util';
	import { invokeCommand } from '$lib/invoke';
	import type { ImportData } from '$lib/models';
	import { activeProfile, refreshProfiles } from '$lib/stores';

	import { confirm, open } from '@tauri-apps/plugin-dialog';
	import { getCurrentWindow } from '@tauri-apps/api/window';
	import { open as shellOpen } from '@tauri-apps/plugin-shell';
	import { writeText } from '@tauri-apps/plugin-clipboard-manager';

	let importR2Open = false;
	let newProfileOpen = false;
	let exportCodePopup: ExportCodePopup;

	let importProfileOpen = false;
	let importProfileData: ImportData | null = null;

	let profileOperation: 'rename' | 'duplicate' = 'rename';
	let profileOperationName = '';
	let profileOperationOpen = false;
	let profileOperationInProgress = false;

	let aboutOpen = false;

	const appWindow = getCurrentWindow();

	async function importLocalMod() {
		let path = await open({
			title: '选择要导入的模组文件',
			filters: [{ name: 'Dll or zip', extensions: ['dll', 'zip'] }]
		});

		if (path === null) return;
		await invokeCommand('import_local_mod', { path });
		await refreshProfiles();
	}

	async function importFile() {
		let path = await open({
			title: '选择要导入的文件',
			filters: [{ name: 'Profile file', extensions: ['r2z'] }]
		});

		if (path === null) return;
		let data = await invokeCommand<ImportData>('import_file', { path });

		importProfileData = data;
		importProfileOpen = true;
	}

	async function exportFile() {
		let dir = await open({
			directory: true,
			title: '选择要将配置文件导出到的目录'
		});

		if (dir === null) return;
		invokeCommand('export_file', { dir });
	}

	async function setAllModsState(enable: boolean) {
		await invokeCommand('set_all_mods_state', { enable });
		activeProfile.update((profile) => profile);
	}

	function openProfileOperation(operation: 'rename' | 'duplicate') {
		profileOperation = operation;
		profileOperationName = $activeProfile?.name ?? 'Unknown';
		profileOperationOpen = true;
	}

	async function doProfileOperation() {
		if (profileOperationInProgress) return;

		profileOperationInProgress = true;

		try {
			if (profileOperation == 'rename') {
				await invokeCommand('rename_profile', { name: profileOperationName });
			} else if (profileOperation == 'duplicate') {
				await invokeCommand('duplicate_profile', { name: profileOperationName });
			}
		} catch (e) {
			profileOperationInProgress = false;
			throw e;
		}

		await refreshProfiles();
		profileOperationInProgress = false;
		profileOperationOpen = false;
	}

	async function uninstallDisabledMods() {
		let confirmed = await confirm('您确定要卸载所有禁用的模组吗?');
		if (!confirmed) return;

		await invokeCommand<number>('remove_disabled_mods');
		await refreshProfiles();
	}

	async function zoom(value: { delta: number } | { factor: number }) {
		await invokeCommand('zoom_window', { value });
	}

	async function copyLaunchArgs() {
		let str = await invokeCommand<string>('get_launch_args');
		writeText(str);
	}

	async function clearModCache() {
		let result = await confirm(
			"您确定要删除所有缓存的模组吗？这可能会导致已安装的模组占用更多的磁盘空间。只有在您了解相关情况时才继续操作!"
		);

		if (!result) return;

		await invokeCommand('clear_download_cache', { soft: false });
	}

	const hotkeys: { [key: string]: () => void } = {
		'+': () => zoom({ delta: 0.25 }),
		'-': () => zoom({ delta: -0.25 }),
		'0': () => zoom({ factor: 1 }),
		n: () => (newProfileOpen = true),
		d: () => openProfileOperation('duplicate')
	};

	onMount(() => {
		document.onkeydown = ({ key, ctrlKey }) => {
			if (key === 'F2') {
				openProfileOperation('rename');
				return;
			}

			if (!ctrlKey) return;

			const hotkey = hotkeys[key];
			if (hotkey !== undefined) hotkey();
		};
	});
</script>

<header data-tauri-drag-region class="flex h-8 flex-shrink-0 bg-slate-800">
	<Menubar.Root class="flex items-center py-1">
		<img src="favicon.png" alt="Gale logo" class="ml-4 mr-2 h-5 w-5 opacity-50" />
		<MenubarMenu label="文件">
			<MenubarItem
				on:click={() => invokeCommand('open_profile_dir')}
				text="打开配置文件目录"
			/>
			<MenubarItem on:click={() => invokeCommand('open_game_dir')} text="打开游戏目录" />
			<MenubarSeparator />
			<MenubarItem on:click={() => invokeCommand('open_game_log')} text="打开游戏日志" />
			<MenubarItem on:click={() => invokeCommand('open_gale_log')} text="打开 Gale 日志" />
			<MenubarSeparator />
			<MenubarItem on:click={clearModCache} text="清除模组缓存" />
			<MenubarItem
				on:click={() => invokeCommand('clear_download_cache', { soft: true })}
				text="清除未使用的模组缓存"
			/>
			<MenubarItem on:click={() => invokeCommand('trigger_mod_fetch')} text="获取模组" />
		</MenubarMenu>
		<MenubarMenu label="配置">
			<MenubarItem
				on:click={() => (newProfileOpen = true)}
				text="创建新的配置文件"
				key="Ctrl N"
			/>
			<MenubarItem
				on:click={() => openProfileOperation('rename')}
				text="重命名当前配置文件"
				key="F2"
			/>
			<MenubarItem
				on:click={() => openProfileOperation('duplicate')}
				text="重复当前配置文件"
				key="Ctrl D"
			/>
			<MenubarSeparator />
			<MenubarItem on:click={() => invokeCommand('copy_dependency_strings')} text="复制模组列表" />
			<MenubarItem on:click={() => invokeCommand('copy_debug_info')} text="复制调试信息" />
			<MenubarItem on:click={copyLaunchArgs} text="复制启动参数" />
			<MenubarSeparator />
			<MenubarItem on:click={() => setAllModsState(true)} text="启用所有模组" />
			<MenubarItem on:click={() => setAllModsState(false)} text="禁用所有模组" />
			<MenubarItem on:click={uninstallDisabledMods} text="卸载禁用模组" />
		</MenubarMenu>
		<MenubarMenu label="导入">
			<MenubarItem on:click={() => (importProfileOpen = true)} text="...来自代码配置" />
			<MenubarItem on:click={importFile} text="...来自文件配置" />
			<MenubarItem on:click={importLocalMod} text="...本地模组" />
			<MenubarItem on:click={() => (importR2Open = true)} text="...来自r2modman配置" />
		</MenubarMenu>
		<MenubarMenu label="导出">
			<MenubarItem on:click={() => exportCodePopup.open()} text="...配置文件作为代码" />
			<MenubarItem on:click={exportFile} text="...配置作为文件" />
		</MenubarMenu>
		<MenubarMenu label="窗口">
			<MenubarItem
				on:click={() => invokeCommand('zoom_window', { value: { delta: 0.25 } })}
				text="放大"
				key="Ctrl +"
			/>
			<MenubarItem
				on:click={() => invokeCommand('zoom_window', { value: { delta: -0.25 } })}
				text="缩小"
				key="Ctrl -"
			/>
			<MenubarItem
				on:click={() => invokeCommand('zoom_window', { value: { factor: 1 } })}
				text="重置"
				key="Ctrl 0"
			/>
		</MenubarMenu>
		<MenubarMenu label="帮助">
			<MenubarItem
				on:click={() => shellOpen('https://github.com/Kesomannen/ModManager/issues/')}
				text="报告错误"
			/>
			<MenubarItem
				on:click={() => shellOpen('https://discord.gg/sfuWXRfeTt')}
				text="加入discord服务器"
			/>
			<MenubarItem on:click={() => (aboutOpen = true)} text="关于 Gale" />
		</MenubarMenu>
	</Menubar.Root>

	<Button.Root class="group ml-auto px-3 py-1.5 hover:bg-slate-700" on:click={appWindow.minimize}>
		<Icon icon="mdi:minimize" class="text-slate-500 group-hover:text-white" />
	</Button.Root>
	<Button.Root class="group px-3 py-1.5 hover:bg-slate-700" on:click={appWindow.toggleMaximize}>
		<Icon icon="mdi:maximize" class="text-slate-500 group-hover:text-white" />
	</Button.Root>
	<Button.Root class="group px-3 py-1.5 hover:bg-red-700" on:click={appWindow.close}>
		<Icon icon="mdi:close" class="text-slate-500 group-hover:text-white" />
	</Button.Root>
</header>

<Popup
	title="{profileOperation === 'rename' ? '重命名' : '复制'}配置"
	canClose={!profileOperationInProgress}
	bind:open={profileOperationOpen}
>
	<p class="mb-1 text-slate-300">
		{profileOperation == 'duplicate'
			? '输入重复配置文件的名称:'
			: '输入配置文件的新名称:'}
	</p>
	<InputField
		bind:value={profileOperationName}
		placeholder="输入名称..."
		size="lg"
		class="w-full"
		on:submit={doProfileOperation}
	/>
	{#if profileOperation == 'duplicate'}
		<p class="mt-2 text-sm text-slate-400">
			这过程可能需要最多一分钟，具体取决于配置的大小，请耐心等待。
		</p>
	{/if}
	<div class="ml-auto mt-2 flex justify-end gap-2">
		{#if !profileOperationInProgress}
			<BigButton color="slate" on:click={() => (profileOperationOpen = false)}>取消</BigButton>
		{/if}
		<BigButton
			color="accent"
			fontWeight="medium"
			disabled={profileOperationInProgress}
			on:click={doProfileOperation}
		>
			{#if profileOperationInProgress}
				<Icon icon="mdi:loading" class="my-1 animate-spin text-lg" />
			{:else}
				{profileOperation === 'rename' ? '重命名' : '复制'}
			{/if}
		</BigButton>
	</div>
</Popup>

<AboutPopup bind:open={aboutOpen} />
<ImportR2Popup bind:open={importR2Open} />
<NewProfilePopup bind:open={newProfileOpen} />
<ExportCodePopup bind:this={exportCodePopup} />
<ImportProfilePopup bind:open={importProfileOpen} bind:data={importProfileData} />
