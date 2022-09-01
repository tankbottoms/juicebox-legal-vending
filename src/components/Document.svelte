<script lang="ts">
	import { writable } from 'svelte/store';
	import IconCaret from './Icons/IconCaret.svelte';
	import './document.css';
	import IconDoubleCaret from './Icons/IconDoubleCaret.svelte';

	export let path: string;
	export let sectionRef: HTMLElement;
	export let dog: string;
	let rendered: any;
	let metadata: any;
	let variables: any;
	let contentRef: any;

	function scrollDown() {
		sectionRef.scrollTop += window.innerHeight - 100;
	}

	function scrollUp() {
		sectionRef.scrollTop -= window.innerHeight - 100;
	}

	function scrollToBottom() {
		sectionRef.scrollTop = sectionRef.scrollHeight;
	}

	function scrollToTop() {
		sectionRef.scrollTop = 0;
	}

	async function loadData(path: string) {
		// NOTE: The dynamic import *must* have an extension, otherwise it won't load.
		// Keep this in mind for svx files, i.e. mixed svelte and markdown.
		const pathWithoutExtension = path.substring(1).replace(/\.[^/.]+$/, '');
		const globs = import.meta.glob('../docs/**/*.{md,svx}');
		// NOTE: this worked locally await import('..docs/' + pathWithoutExtension + '.md');
		// not below i.e. the string literal, but above, the import.meta.glob - works in production.
		// Looks insane.
		let imported: any;
		let importedVariables: any;
		try {
			imported = await globs?.[`../docs/${pathWithoutExtension}.md`]();
			rendered = imported.default;
		} catch (e) {
			console.info('No markdown file found for ' + pathWithoutExtension);
		}
		try {
			importedVariables = await globs?.[`../docs/${pathWithoutExtension}.json`]();
			metadata = imported.metadata || {};
			// TODO: add a value key to the importedVariables object.
			variables = writable(importedVariables.default);
		} catch (e) {
			// console.info('No variables found for this page.');
		}
	}

	async function resetDocument() {
		// TODO: Dry this out, it's copied from above
		rendered = undefined;
		const pathWithoutExtension = path.substring(1).replace(/\.[^/.]+$/, '');
		const globs = import.meta.glob('../docs/**/*.{md,svx,json}');
		// NOTE: this worked locally await import('..docs/' + pathWithoutExtension + '.md');
		// not below i.e. the string literal, but above, the import.meta.glob - works in production.
		// Looks insane.
		let imported: any;
		try {
			imported = await globs?.[`../docs/${pathWithoutExtension}.md`]();
		} catch (e) {
			console.log(e);
		}
		rendered = imported.default;
	}

	async function updateVariables() {
		await resetDocument();
		if (contentRef) {
			$variables.forEach((variable: any) => {
				if (variable.default) {
					contentRef.innerHTML = contentRef.innerHTML.replace(
						new RegExp(`\\[${variable.name}\\]`, 'g'),
						variable.default
					);
				}
			});
		}
	}

	$: {
		loadData(path);
	}
</script>

<article bind:this={contentRef}>
	{#if rendered}
		<div id="content">
			<svelte:component this={rendered} />
		</div>
	{/if}
	<footer>
		{#if sectionRef?.scrollTop > 100}
			<span on:click={scrollToTop}>
				<IconDoubleCaret circle width="17" height="17" />
			</span>
			<span on:click={scrollUp}>
				<IconCaret circle transform="rotate(180)" />
			</span>
		{/if}
		<span on:click={scrollDown}>
			<IconCaret circle />
		</span>
		<span on:click={scrollToBottom}>
			<IconDoubleCaret circle width="17" height="17" transform="rotate(180)" />
		</span>
	</footer>
</article>

<style>
	article {
		position: relative;
	}

	footer {
		position: sticky;
		bottom: 0;
		display: flex;
		justify-content: center;
		padding: 0.7rem;
		background-color: var(--background-l1);
	}

	footer span {
		margin: 0 0.5rem;
	}

	#content {
		max-width: 850px;
		padding: 0rem 2rem;
		word-break: break-word;
	}
</style>