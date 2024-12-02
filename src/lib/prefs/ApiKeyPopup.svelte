<script context="module" lang="ts">
	export const apiKeyPopupOpen = writable(false);
</script>

<script lang="ts">
	import BigButton from '$lib/components/BigButton.svelte';
	import ConfirmPopup from '$lib/components/ConfirmPopup.svelte';
	import InputField from '$lib/components/InputField.svelte';
	import Link from '$lib/components/Link.svelte';
	import { invokeCommand } from '$lib/invoke';
	import { Button } from 'bits-ui';

	import { writable } from 'svelte/store';

	let token: string;

	async function submit() {
		if (token.length == 0) {
			await invokeCommand('clear_thunderstore_token');
		} else {
			await invokeCommand('set_thunderstore_token', { token });
			token = '';
		}

		$apiKeyPopupOpen = false;
	}
</script>

<ConfirmPopup title="设置 Thunderstore API 令牌" bind:open={$apiKeyPopupOpen}>
    <p>
        在下方输入您的 Thunderstore API 令牌，或者留空以清除当前的令牌。此令牌用于将 modpack 发布到 Thunderstore，并会在您的计算机上安全存储。
    </p>

    <p class="mb-3 mt-2">
        设置后，您将 <b>无法</b> 再次查看该令牌。
    </p>

    <InputField
        placeholder="输入 API 令牌..."
        class="w-full"
        on:submit={submit}
        bind:value={token}
    />

    <details>
        <summary class="mt-1 cursor-pointer text-sm text-slate-400">
            不确定如何获取您的 API 令牌？
        </summary>
        <ol class="my-1 ml-1 flex flex-col gap-1">
            <li>
                1. 登录 <Link href="https://thunderstore.io/">thunderstore.io</Link>。
            </li>

            <li>
                2. 转到 <Link href="https://thunderstore.io/settings/teams/">Teams</Link>。
            </li>

            <li>
                3. 如果您尚未创建，请创建一个新团队。团队名称应为您的用户名。
            </li>

            <li>
                4. 选择一个团队并转到左侧边栏的 <code>Service Accounts</code>。
            </li>

            <li>
                5. 点击 <code>添加服务账户</code>，并选择一个合适的昵称，例如 "gale"。
            </li>

            <li>
                6. 提交后，将显示 API 令牌。请确保将其复制并粘贴到此处，因为一旦离开页面，您将无法再次看到它。
            </li>
        </ol>

        <b>
            请勿与他人分享令牌，因为它允许使用您的名义更新、发布或删除软件包！
        </b>
    </details>

    <svelte:fragment slot="buttons">
        <BigButton color="accent" fontWeight="medium" on:click={submit}>提交</BigButton>
    </svelte:fragment>
</ConfirmPopup>
