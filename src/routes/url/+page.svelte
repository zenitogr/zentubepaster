<script lang="ts">
	import { page } from '$app/stores';
	import { Downloader } from '$lib/components/ui';
	import { invoke } from '@tauri-apps/api/tauri';
	import { onMount,afterUpdate } from 'svelte';
	import MaterialSymbolsRefresh from '~icons/material-symbols/refresh';
	/*import Listener from '../listener.svelte'; */
	
	let invoking: Promise<YouTube.Video>;

	afterUpdate(() => {
		const id = $page.url.searchParams.get('v')!;
		invoking = invoke<YouTube.Video>('get_info', { id });
	});
</script>
{#await invoking}
	<div class="flex justify-center">
		<MaterialSymbolsRefresh class="m-8 h-8 w-8 animate-spin" />
	</div>
{:then video}
	{#if video}
		<Downloader {video} />
	{/if}
{:catch _}
	<div class="p-4 text-red-400">Invalid video</div>
{/await}
