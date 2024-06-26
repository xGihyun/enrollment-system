<script lang="ts">
	import type { SubjectDetails } from '$lib/types/subject';
	import { createRender, createTable, Render, Subscribe } from 'svelte-headless-table';
	import { addSelectedRows } from 'svelte-headless-table/plugins';
	import { readable, type Writable } from 'svelte/store';
	import * as Table from '$lib/components/ui/table';
	import TableCheckbox from './table-checkbox.svelte';
	import TableActions from './table-actions.svelte';
	import { getSelectedRowData } from '$lib';

	export let data: SubjectDetails[];
	export let selectedRows: Writable<string[]>;

	$: table = createTable(readable(data), {
		select: addSelectedRows()
	});

	$: columns = table.createColumns([
		table.column({
			accessor: 'subject_id',
			header: (_, { pluginStates }) => {
				const { allPageRowsSelected } = pluginStates.select;
				return createRender(TableCheckbox, {
					checked: allPageRowsSelected
				});
			},
			cell: ({ row }, { pluginStates }) => {
				const { getRowState } = pluginStates.select;
				const { isSelected } = getRowState(row);

				return createRender(TableCheckbox, {
					checked: isSelected
				});
			}
		}),
		table.column({
			accessor: ({ subject_id }) => {
				return subject_id;
			},
			header: 'ID'
		}),
		table.column({
			accessor: 'subject_name',
			header: 'Name'
		}),
		table.column({
			accessor: 'year_levels',
			header: 'Year Levels',
			cell: ({ value }) => {
				return `${value.length} Year Levels`;
			}
		}),
		table.column({
			accessor: (subject) => subject,
			header: '',
			cell: ({ value }) => {
				return createRender(TableActions, { subject: value });
			}
		})
	]);

	$: ({ headerRows, pageRows, tableAttrs, tableBodyAttrs, pluginStates } =
		table.createViewModel(columns));
	$: ({ selectedDataIds } = pluginStates.select);

	$: getSelectedRowData(data, selectedRows, $selectedDataIds, 'subject_id');
</script>

<Table.Root {...$tableAttrs}>
	<Table.Header>
		{#each $headerRows as headerRow}
			<Subscribe rowAttrs={headerRow.attrs()}>
				<Table.Row>
					{#each headerRow.cells as cell (cell.id)}
						<Subscribe attrs={cell.attrs()} let:attrs props={cell.props()}>
							<Table.Head {...attrs}>
								<Render of={cell.render()} />
							</Table.Head>
						</Subscribe>
					{/each}
				</Table.Row>
			</Subscribe>
		{/each}
	</Table.Header>
	<Table.Body {...$tableBodyAttrs}>
		{#each $pageRows as row (row.id)}
			<Subscribe rowAttrs={row.attrs()} let:rowAttrs>
				<Table.Row {...rowAttrs}>
					{#each row.cells as cell (cell.id)}
						<Subscribe attrs={cell.attrs()} let:attrs>
							<Table.Cell {...attrs}>
								<Render of={cell.render()} />
							</Table.Cell>
						</Subscribe>
					{/each}
				</Table.Row>
			</Subscribe>
		{/each}
	</Table.Body>
</Table.Root>
