<script lang="ts">
	import { SvelteComponent, onMount } from 'svelte';
	import {
		Button,
		IconButton,
		Modal,
		ScrollbarContainer,
		Separator,
		Switch,
		Tooltip
	} from '$lib/components';
	import { storage } from '$lib/features';
	import { defaultPriorities } from '$lib/helpers';
	import { PlusIcon, PriorityIcon } from '$lib/icons';
	import { settings } from '$lib/stores';
	import type { Priority } from '$lib/types';
	import PriorityItem from './PriorityItem.svelte';

	let editing = false;
	let mounted = false;
	let priorities: Priority[] = [];
	let scrollContainer: SvelteComponent;

	function handleSave(e: CustomEvent<Priority>) {
		priorities.push(e.detail);
		priorities.sort((a, b) => b.value - a.value);
		priorities = priorities;

		editing = false;
	}

	function handleDelete(criteria: Priority['criteria']) {
		priorities = priorities.filter((priority) => priority.criteria !== criteria);
	}

	onMount(() => {
		priorities = storage.get('priorities') || defaultPriorities;
	});

	$: if (mounted) {
		storage.set('priorities', priorities);
		dispatchEvent(new CustomEvent('refetch'));
	} else if (priorities.length) {
		mounted = true;
	}

	$: if (editing) {
		setTimeout(() => {
			scrollContainer?.scrollTo({ top: 999, behavior: 'smooth' });
		}, 10);
	}
</script>

<Modal title="Priorities" on:close={() => (editing = false)}>
	<Tooltip slot="trigger" content="Priorities" hover position="bottom">
		<IconButton large rounded>
			<PriorityIcon />
		</IconButton>
	</Tooltip>

	<ScrollbarContainer slot="content" margin="2rem 1rem" bind:this={scrollContainer}>
		<div class="content">
			<Switch label="Enable priority sorting" bind:active={$settings.prioritySorting} />
			<p class="paragraph">
				By enabling priority sorting and specifying priority criterias, you can customize the
				importance of notifications and manage them in your own way.
			</p>
			<Switch label="Show priorities on notifications" bind:active={$settings.showPriority} />
			<div class="buttons">
				<Button small on:click={() => (priorities = defaultPriorities)}>Reset to default</Button>
				<Button secondary small on:click={() => (priorities = [])}>Clear</Button>
			</div>
			<Separator marginY={1} />
			{#each priorities as priority}
				<PriorityItem
					{priorities}
					item={priority}
					on:delete={() => handleDelete(priority.criteria)}
				/>
			{/each}
			{#if editing}
				<PriorityItem {priorities} editing on:save={handleSave} on:exit={() => (editing = false)} />
			{/if}
			{#if !priorities.length && !editing}
				<p class="paragraph">No priority criterias yet.</p>
			{/if}
			<Button secondary on:click={() => (editing = true)}>
				<PlusIcon />
				Add a new priority criteria
			</Button>
		</div>
	</ScrollbarContainer>
</Modal>

<style lang="scss">
	.content {
		display: flex;
		flex-direction: column;
		padding: 2rem;
		gap: 1rem;
	}

	.paragraph {
		@include typography.base;

		color: variables.$grey-4;
	}

	.buttons {
		display: flex;
		gap: 1rem;
	}
</style>
