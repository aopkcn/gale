<script lang="ts">
	import SearchBar from '$lib/components/SearchBar.svelte';
	import { activeGame, games, setActiveGame } from '$lib/stores';
	import Icon from '@iconify/svelte';
	import { Button } from 'bits-ui';
	import { invokeCommand } from '$lib/invoke';
	import Link from './Link.svelte';
	import { ModLoader } from '$lib/models';
	import Tooltip from './Tooltip.svelte';
	import { titleCase } from '$lib/util';

	export let onSelect: () => void;

	let shownGames = games;
	let searchTerm = '';

	$: refresh(searchTerm);

	function refresh(searchTerm: string) {
		let lowerSearch = searchTerm.toLowerCase();

		let newGames =
			searchTerm.length > 0
				? games.filter((game) => {
						return game.name.toLowerCase().includes(lowerSearch);
					})
				: games;

		newGames.sort((a, b) => {
			if (searchTerm.length === 0) {
				if (a.favorite && !b.favorite) return -1;
				if (!a.favorite && b.favorite) return 1;

				if (a.popular && !b.popular) return -1;
				if (!a.popular && b.popular) return 1;
			}

			return a.name.localeCompare(b.name);
		});

		shownGames = newGames;
	}
</script>

<div class="relative mt-1 flex-grow">
	<SearchBar bind:value={searchTerm} placeholder="搜索游戏..." />
</div>

<div class="mt-2 flex h-80 flex-col overflow-y-scroll">
	{#if shownGames.length > 0}
		{#each shownGames as game}
			<Button.Root
				class="group mr-2 flex items-center rounded-lg border border-slate-500 p-1.5 hover:bg-slate-700 {$activeGame?.slug ===
				game.slug
					? 'bg-slate-700'
					: 'border-opacity-0 hover:bg-slate-700'}"
				on:click={() => {
					setActiveGame(game);
					onSelect();
				}}
			>
				<img src="games/{game.slug}.webp" alt={game.name} class="mr-2 size-12 rounded" />

				<div class="flex-grow pl-1 text-left">
					<div class="font-medium text-white">
						{game.name}
					</div>

					<div class="text-slate-400">
						<span>{game.modLoader} </span>

						{#if game.modLoader !== ModLoader.BepInEx}
							<Tooltip
								class="inline-flex rounded bg-red-600 p-0.5 text-sm text-white"
								text="实验性支持，存在风险!"
							>
								<Icon icon="mdi:beta" />
							</Tooltip>
						{/if}

						{#if game.platforms.length > 0}
							<span class="mx-1 text-slate-500">|</span>

							<span class="mr-1">{game.platforms.map(titleCase).join(', ')}</span>
						{/if}
					</div>
				</div>

				<Button.Root
					class="mr-1 rounded p-1.5 hover:bg-slate-600 {game.favorite
						? 'block'
						: 'hidden group-hover:block'}"
					on:click={(evt) => {
						evt.stopPropagation();
						game.favorite = !game.favorite;
						refresh(searchTerm);
						invokeCommand('favorite_game', { slug: game.slug });
					}}
				>
					<Icon
						icon={game.favorite ? 'mdi:star' : 'mdi:star-outline'}
						class="text-xl text-accent-500"
					/>
				</Button.Root>
			</Button.Root>
		{/each}
	{:else}
		<div class="mt-4 text-center text-slate-300">未找到游戏 😢</div>
		<div class="max-w-[35rem] text-sm text-slate-400">
		  找不到您的游戏？如果游戏是新加入 Thunderstore 的，我们可能还未添加它。
		  如果您在
		  <Link href="https://thunderstore.io">thunderstore.io</Link>
		  上能找到它，但在这里找不到，请通过
		  <Link href="https://discord.com/channels/1168655651455639582/1246088342458863618">Discord</Link>
		  与我们联系，或者在
		  <Link href="https://github.com/Kesomannen/ModManager/issues/">我们的 Github</Link>
		  上提交问题。
		</div>
	  {/if}
</div>
