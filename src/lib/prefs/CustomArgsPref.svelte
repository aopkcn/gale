<script lang="ts">
	import Checkbox from '$lib/components/Checkbox.svelte';
	import InputField from '$lib/components/InputField.svelte';
	import Label from '$lib/components/Label.svelte';
	import Icon from '@iconify/svelte';
	import { Button } from 'bits-ui';

	export let value: string[] | null;
	export let set: (value: string[] | null) => Promise<void>;

	let newArg = '';
</script>

<div class="mt-1 flex items-center">
	<Label text="设置自定义启动参数">
		<p>
			允许您将自定义参数添加到启动命令中。根据 <b>启动模式</b>，
			这些参数将针对游戏或 Steam 可执行文件运行。
		</p>
	
		<p>
			每个条目只能传递一个参数，因此不要传递 <code>--foo value</code>，
			而是分别传递 <code>--foo</code> 和 <code>value</code>。
		</p>
	</Label>
	

	<Checkbox
		value={value !== null}
		onValueChanged={(newValue) => {
			set(newValue ? [] : null);
		}}
	/>
</div>

{#if value !== null}
	<div class="mt-1 flex flex-col gap-1 pl-[35%] text-slate-300">
		{#each value as argument, i}
			<div class="flex gap-1">
				<Button.Root
					class="rounded-lg p-1.5 text-xl text-slate-400 hover:bg-slate-700 hover:text-slate-300"
					on:click={() => {
						if (value === null) return;
						value = value.filter((_, index) => index !== i);
						set(value);
					}}
				>
					<Icon icon="mdi:remove" />
				</Button.Root>
				<InputField
					class="flex-grow"
					value={argument}
					on:change={({ detail }) => {
						if (value === null) return;
						value[i] = detail;
						set(value);
					}}
				/>
			</div>
		{/each}

		<InputField
			placeholder="输入新参数..."
			bind:value={newArg}
			on:change={() => {
				if (newArg.length === 0 || value === null) return;

				value = [...value, newArg];
				newArg = '';
				set(value);
			}}
		/>
	</div>
{/if}

<style lang="postcss">
	code {
		@apply bg-slate-900 px-1 text-sm;
	}
</style>
