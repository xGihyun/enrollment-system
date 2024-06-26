<script lang="ts">
	import { FormSection } from '$lib/components/forms';
	import { TableSectionLevels } from '$lib/components/tables/admin';
	import * as Card from '$lib/components/ui/card';
	import * as Dialog from '$lib/components/ui/dialog';
	import { Button } from '$lib/components/ui/button';
	import { writable } from 'svelte/store';
	import { deleteData } from '$lib';
	import DeleteIcon from 'virtual:icons/material-symbols/delete-outline';
	import CirclePlusOutline from 'virtual:icons/flowbite/circle-plus-outline';
	import { Pagination } from '$lib/components/index.js';
	import { setContext } from 'svelte';
	import {
		StrandCombobox,
		YearLevelCombobox
	} from '$lib/components/combobox/index.js';

	export let data;

	const selectedRows = writable<string[]>([]);

	setContext('formSection', data.form);
	setContext('yearLevels', data.yearLevels || []);
	setContext('strands', data.strands || []);
	setContext('teachers', data.teachers || []);
</script>

<div class="flex gap-2 justify-between">
	<div class="flex gap-2 w-full flex-col sm:flex-row">
		<YearLevelCombobox yearLevels={data.yearLevels || []} selected={data.selectedYearLevelId} />
		<StrandCombobox strands={data.strands || []} selected={data.selectedStrandId} />
	</div>
</div>

<div class="flex flex-col gap-2 flex-[2_2_0%] h-full">
	<Card.Root>
		<Card.Header class="flex flex-row items-center justify-between space-y-0">
			<div class="space-y-1.5">
				<Card.Title>Sections</Card.Title>
				<Card.Description>Sections and their year levels</Card.Description>
			</div>

			{#if $selectedRows.length > 0}
				<Dialog.Root>
					<Dialog.Trigger asChild let:builder>
						<Button class="space-x-1" builders={[builder]}>
							<DeleteIcon />
							<span>Delete</span>
						</Button>
					</Dialog.Trigger>

					<Dialog.Content>
						<Dialog.Header>
							<Dialog.Title>Delete sections?</Dialog.Title>
							<Dialog.Description>
								This action cannot be undone. This will permanently delete all selected sections and
								it's related data from the server.
							</Dialog.Description>
						</Dialog.Header>
						<Dialog.Footer>
							<Button on:click={() => deleteData($selectedRows, '/api/sections')}>Delete</Button>
						</Dialog.Footer>
					</Dialog.Content>
				</Dialog.Root>
			{:else}
				<Dialog.Root>
					<Dialog.Trigger asChild let:builder>
						<Button class="space-x-1" builders={[builder]}>
							<CirclePlusOutline />
							<span>Create</span>
						</Button>
					</Dialog.Trigger>

					<Dialog.Content>
						<Dialog.Header>
							<Dialog.Title>Create a new section</Dialog.Title>
						</Dialog.Header>

						<FormSection />
					</Dialog.Content>
				</Dialog.Root>
			{/if}
		</Card.Header>
		<Card.Content>
			<TableSectionLevels data={data.sectionLevels?.section_levels || []} {selectedRows} />
		</Card.Content>
	</Card.Root>

	<Pagination count={data.sectionLevels?.count || 0} />
</div>
