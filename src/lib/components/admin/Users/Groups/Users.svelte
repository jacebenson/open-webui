<script lang="ts">
	import { getContext } from 'svelte';
	const i18n = getContext('i18n');

	import Tooltip from '$lib/components/common/Tooltip.svelte';
	import Plus from '$lib/components/icons/Plus.svelte';
	import { WEBUI_BASE_URL } from '$lib/constants';
	import Checkbox from '$lib/components/common/Checkbox.svelte';
	import Badge from '$lib/components/common/Badge.svelte';
	import Search from '$lib/components/icons/Search.svelte';

	export let users = [];
	export let userIds = [];

	let filteredUsers = [];

	$: filteredUsers = users
		.filter((user) => {
			if (query === '') {
				return true;
			}

			return (
				user.name.toLowerCase().includes(query.toLowerCase()) ||
				user.email.toLowerCase().includes(query.toLowerCase())
			);
		})
		.sort((a, b) => {
			const aUserIndex = userIds.indexOf(a.id);
			const bUserIndex = userIds.indexOf(b.id);

			// Compare based on userIds or fall back to alphabetical order
			if (aUserIndex !== -1 && bUserIndex === -1) return -1; // 'a' has valid userId -> prioritize
			if (bUserIndex !== -1 && aUserIndex === -1) return 1; // 'b' has valid userId -> prioritize

			// Both a and b are either in the userIds array or not, so we'll sort them by their indices
			if (aUserIndex !== -1 && bUserIndex !== -1) return aUserIndex - bUserIndex;

			// If both are not in the userIds, fallback to alphabetical sorting by name
			return a.name.localeCompare(b.name);
		});

	let query = '';
</script>

<div>
	<div class="flex w-full">
		<div class="flex flex-1">
			<div class=" self-center mr-3">
				<Search />
			</div>
			<input
				class=" w-full text-sm pr-4 rounded-r-xl outline-hidden bg-transparent"
				bind:value={query}
				placeholder={$i18n.t('Search')}
			/>
		</div>
	</div>

	<div class="mt-3 scrollbar-hidden">
		<div class="flex flex-col gap-2.5">
			{#if filteredUsers.length > 0}
				{#each filteredUsers as user, userIdx (user.id)}
					<div class="flex flex-row items-center gap-3 w-full text-sm">
						<div class="flex items-center">
							<Checkbox
								state={userIds.includes(user.id) ? 'checked' : 'unchecked'}
								on:change={(e) => {
									if (e.detail === 'checked') {
										userIds = [...userIds, user.id];
									} else {
										userIds = userIds.filter((id) => id !== user.id);
									}
								}}
							/>
						</div>

						<div class="flex w-full items-center justify-between">
							<Tooltip content={user.email} placement="top-start">
								<div class="flex">
									<div class=" font-medium self-center">{user.name}</div>
								</div>
							</Tooltip>

							{#if userIds.includes(user.id)}
								<Badge type="success" content="member" />
							{/if}
						</div>
					</div>
				{/each}
			{:else}
				<div class="text-gray-500 text-xs text-center py-2 px-10">
					{$i18n.t('No users were found.')}
				</div>
			{/if}
		</div>
	</div>
</div>
