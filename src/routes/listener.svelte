<script lang="ts">
	import type { UnlistenFn } from '@tauri-apps/api/event';
	import { goto } from '$app/navigation';
	import { onDestroy, onMount } from 'svelte';
	import {
		onClipboardUpdate,
		onImageUpdate,
		onTextUpdate,
		onHTMLUpdate,
		onRTFUpdate,
		onFilesUpdate,
		startListening,
		listenToMonitorStatusUpdate,
		isMonitorRunning,
		hasHTML,
		hasImage,
		hasText,
		hasRTF,
		hasFiles
	} from 'tauri-plugin-clipboard-api';


	let text = '';
	let files: string[] = [];
	let base64Image = '';
	let htmlMonitorContent = '';
	let monitorRunning = false;
	let rtf = '';
	let unlistenTextUpdate: UnlistenFn;
	let unlistenImageUpdate: UnlistenFn;
	let unlistenHtmlUpdate: UnlistenFn;
	let unlistenRTF: UnlistenFn;
	let unlistenClipboard: () => Promise<void>;
	let unlistenFiles: UnlistenFn;
	const has = {
		hasHTML: false,
		hasImage: false,
		hasText: false,
		hasRTF: false,
		hasFiles: false
	};
	onMount(async () => {
		unlistenTextUpdate = await onTextUpdate((newText) => {
			text = newText;
			/* console.log('onTextUpdate -> clipurl = text')
			clipurl = text;
			console.log('init urlloadauto')
			urlloadauto();
			console.log('urlloadauto called') */
			const text2 = text.trim();
		if (text.length) {
			const url = new URL(text2);
			const v = url.searchParams.get('v');
			if (v) {
				goto(`/url?v=${v}`);
			}
		}
		});
		unlistenHtmlUpdate = await onHTMLUpdate((newHtml) => {
			htmlMonitorContent = newHtml;
		});
		unlistenImageUpdate = await onImageUpdate((b64Str) => {
			base64Image = b64Str;
		});
		unlistenFiles = await onFilesUpdate((newFiles) => {
			files = newFiles;
		});
		unlistenRTF = await onRTFUpdate((newRTF) => {
			rtf = newRTF;
		});
		unlistenClipboard = await startListening();

		onClipboardUpdate(async () => {
			has.hasHTML = await hasHTML();
			has.hasImage = await hasImage();
			has.hasText = await hasText();
			has.hasRTF = await hasRTF();
			has.hasFiles = await hasFiles();
			console.log('plugin:clipboard://clipboard-monitor/update event received');
		});

		// setInterval(async () => {
		// 	console.log("Running:", await isMonitorRunning());
		// }, 1000);
	});

	listenToMonitorStatusUpdate((running) => {
		monitorRunning = running;
	});

	onDestroy(() => {
		if (unlistenTextUpdate) unlistenTextUpdate();
		if (unlistenImageUpdate) unlistenImageUpdate();
		if (unlistenHtmlUpdate) unlistenHtmlUpdate();
		if (unlistenFiles) unlistenFiles();
		if (unlistenClipboard) unlistenClipboard();
	});
	
</script>

<!-- 
<p><strong>Is Monitor Running:</strong> {monitorRunning}</p>
{#if monitorRunning}
	<button
		class="btn variant-filled-error"
		on:click={async () => {
			await unlistenClipboard();
		}}>Stop Listening</button
	>
{:else}
	<button
		class="btn variant-filled-success"
		on:click={async () => {
			unlistenClipboard = await startListening();
		}}>Start Listening</button
	>
{/if}
<div>
	<div><strong>hasHTML:</strong> {has.hasHTML}</div>
	<div><strong>hasImage:</strong> {has.hasImage}</div>
	<div><strong>hasText:</strong> {has.hasText}</div>
	<div><strong>hasRTF:</strong> {has.hasRTF}</div>
	<div><strong>hasFiles:</strong> {has.hasFiles}</div>
</div>
<div>
	<h2>Clipboard Text</h2>
	{#if text}
		<pre class="border p-2 rounded-lg">{text}</pre>
	{/if}

	<h2>Clipboard Files</h2>
	{#if files}
		<ol class="list-decimal pl-6">
			{#each files as file}
				<li><code>{file}</code></li>
			{/each}
		</ol>
	{/if}
	<h2>Clipboard HTML</h2>
	{#if htmlMonitorContent}
		<div class="border p-2 rounded-lg">{@html htmlMonitorContent}</div>
	{/if}
	<h2>Clipboard RTF</h2>
	{#if rtf}
		<textarea class="w-full h-32 text-black" value={rtf} />
		
	{/if}
	<h2>Clipboard Image</h2>
	{#if base64Image}
		<img width="300" src={`data:image/png;base64, ${base64Image}`} alt="" />
	{/if}
</div> -->