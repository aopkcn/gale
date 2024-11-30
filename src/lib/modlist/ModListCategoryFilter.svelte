<script lang="ts">
	import Dropdown from '$lib/components/Dropdown.svelte';
	import { categories } from '$lib/stores';
	import Icon from '@iconify/svelte';
	import { Button, Select } from 'bits-ui';

	export let selected: string[];
	export let excluded: string[];
	export let icon: string;
	export let label: string;
	const categoryMap = {
    'Ashlands Update': '阿什兰更新',
    'Audio': '音频',
    'Bog Witch Update': '沼泽女巫更新',
    'Building': '建筑',
    'Client-side': '客户端',
    'Crafting': '制作',
    'Enemies': '敌人',
    'Gear': '装备',
    "Hildir's Request Update": '希尔迪的请求更新',
    'Language': '语言',
    'Libraries': '库',
    'Misc': '杂项',
    'Mistlands Update': '迷雾之地更新',
    'Modpacks': '模组包',
    'Mods': '模组',
    'NPCs': 'NPC',
    'PvP': '玩家对玩家',
    'Server-side': '服务器端',
    'Skins': '皮肤',
    'Tools': '工具',
    'Transportation': '交通工具',
    'Tweaks': '调整',
    'Utility': '实用工具',
    'Vehicles': '车辆',
    'World Generation': '世界生成'
  };
</script>

<Dropdown
	items={$categories
		.map(({ name }) => name)
		.filter((category) => !excluded.includes(category))
		.toSorted()}
	multiple={true}
	bind:selected
>
	<Select.Trigger
		let:open
		slot="trigger"
		class="flex flex-grow-[3] basis-0 items-center overflow-hidden rounded-lg border border-slate-500 border-opacity-0 bg-slate-900 px-3 py-1.5 hover:border-opacity-100"
	>
		<Icon class="mr-2 flex-shrink-0 text-lg text-slate-400" {icon} />
		{#if selected.length === 0}
			<span class="truncate text-slate-300">{label}</span>
		{:else}
			<div class="mr-2 flex flex-wrap gap-1">
				{#each selected as category}
					<div
						class="overflow-hidden rounded-lg bg-slate-800 py-0.5 pl-2 pr-0.5 text-sm text-slate-200"
					>
						<span class="overflow-hidden truncate">{categoryMap[category] ||category}</span>
						
						<Button.Root
							class="ml-0.5 rounded-lg px-1.5 hover:bg-slate-700"
							on:click={(evt) => {
								evt.stopPropagation();
								selected = selected.filter((cat) => cat !== category);
							}}
						>
							x
						</Button.Root>
					</div>
				{/each}
			</div>
		{/if}
		<Icon
			class="ml-auto flex-shrink-0 origin-center transform text-lg text-slate-400 transition-all duration-100 ease-out {open
				? 'rotate-180'
				: 'rotate-0'}"
			icon="mdi:chevron-down"
		/>
	</Select.Trigger>
</Dropdown>
