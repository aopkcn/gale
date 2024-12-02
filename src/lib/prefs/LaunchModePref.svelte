<script lang="ts">
	import Label from '$lib/components/Label.svelte';
	import Dropdown from '$lib/components/Dropdown.svelte';
	import InputField from '$lib/components/InputField.svelte';

	import type { LaunchMode } from '$lib/models';
	import { sentenceCase } from '$lib/util';
	import { activeGame } from '$lib/stores';

	export let value: LaunchMode;
	export let set: (value: LaunchMode) => Promise<void>;

	let instances = value.content?.instances ?? 1;
	let intervalSecs = value.content?.intervalSecs ?? 10;

	async function onSelectedChange(newValue: string) {
		value.type = newValue as 'launcher' | 'direct';
		await submit();
	}

	async function submit() {
		if (value.type === 'direct') {
			value.content = { instances, intervalSecs };
		} else {
			value.content = undefined;
		}

		await set(value);
	}

	$: platforms = $activeGame?.platforms ?? [];
</script>

<div class="flex items-center">
	<Label text="启动模式">
		<p>决定游戏的启动方式。</p>
		<p class="my-1.5">
			<b>launcher：</b> 通过指定的平台启动游戏。
		</p>
		<p>
			<b>direct：</b> 直接从可执行文件启动游戏。允许同时启动多个实例。
		</p>
	</Label>

	<Dropdown
		class="flex-grow"
		items={['launcher', 'direct']}
		getLabel={sentenceCase}
		selected={value?.type ?? 'direct'}
		multiple={false}
		disabled={platforms.length === 0}
		{onSelectedChange}
	/>
</div>

<div class="flex items-center">
	<Label text="实例数量">
		一次启动多少个游戏实例。仅在直接模式下可用。
	</Label>

	<InputField
		disabled={value.type !== 'direct'}
		value={instances.toString()}
		on:change={({ detail }) => {
			instances = parseInt(detail);
			submit();
		}}
	/>
</div>

<div class="flex items-center">
	<Label text="启动间隔时间">
		每个实例启动之间等待多少秒。仅在直接模式下并且有多个实例时适用。
	</Label>

	<InputField
		disabled={value.type !== 'direct' || instances <= 1}
		value={intervalSecs.toString()}
		on:change={({ detail }) => {
			intervalSecs = parseInt(detail);
			submit();
		}}
	/>
</div>

