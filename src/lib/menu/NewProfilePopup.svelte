<script lang="ts">
	import { refreshProfiles } from '$lib/stores';
	import { invokeCommand } from '$lib/invoke';
	import BigButton from '$lib/components/BigButton.svelte';
	import InputField from '$lib/components/InputField.svelte';
	import ConfirmPopup from '$lib/components/ConfirmPopup.svelte';

	export let open = false;

	let name: string;

	$: if (open) name = '';

	async function createProfile() {
		await invokeCommand('create_profile', { name });
		refreshProfiles();
		open = false;
	}
</script>

<ConfirmPopup title="创建新的配置文件" bind:open>
	为新配置文件选择一个名称:
	<InputField
		placeholder="输入名称..."
		class="mt-1 w-full"
		on:submit={createProfile}
		bind:value={name}
	/>
	<svelte:fragment slot="buttons">
		<BigButton on:click={createProfile}>创建</BigButton>
	</svelte:fragment>
</ConfirmPopup>
