<script lang="ts">
	import { connection, config, editMode, templates, lang } from '$lib/Stores';
	import { marked } from 'marked';
	import { openModal } from 'svelte-modals';
	import type { TemplateItem } from '$lib/Types';

	export let sel: TemplateItem;
	export let demo = false;

	let unsubscribe: () => void;
	let id = sel?.id;
	let error = false;

	$: template = sel?.template;

	$: if ($config?.state === 'RUNNING' && template) {
		renderTemplate(template);
	}
	/**
	 * Renders template by id to `$templates`
	 */
	async function renderTemplate(data: string) {
		if (!$connection) return;

		const handleResponse = (response: { result?: string }) => {
			if (response?.result && id) {
				$templates[id] = marked.parse(response.result) as any;
				error = false;
			}
		};

		const handleError = (err: any) => {
			if (err?.code === 'template_error' && id) {
				console.warn('render_template', err);
				$templates[id] = err.message;
				error = true;
			}
		};

		try {
			unsubscribe = await $connection.subscribeMessage(handleResponse, {
				type: 'render_template',
				template: data
			});
		} catch (error) {
			handleError(error);
		}
	}
	/**
	 * Handle click
	 */
	async function handleClick() {
		if ($editMode) {
			openModal(() => import('$lib/Modal/TemplateConfig.svelte'), { sel });
		}
	}
</script>

<div
	id="markdown"
	on:click={handleClick}
	on:keydown
	role="button"
	tabindex="0"
	class="container"
>
	{#if demo}
		<div class="template">
			<span>&#123;&#123;</span> template <span>&#125;&#125;</span>
		</div>
	{:else if id && $templates?.[id]}
		<span class:error>{@html $templates?.[id]}</span>
	{:else if template}
		{$lang('unknown')}
	{:else}
		{$lang('template')}
	{/if}
</div>

<style>
	.container {
		--container-padding: 0.72rem;

		/* width: 14.5rem; */
		background-color: var(--theme-button-background-color-off);
		border-radius: 0.65rem;
		margin: 0;
	}

	.container span {
		padding: var(--container-padding);
		display: block;
	}

	/* Phone and Tablet (portrait) */
	@media all and (max-width: 768px) {
		.container {
			width: calc(50vw - 1.45rem);
		}
	}
</style>
