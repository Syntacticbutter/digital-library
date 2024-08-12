<script lang="ts">
	import { onMount } from 'svelte';
	import axios from 'axios';
	import { Button } from '$lib/components/ui/button/index.js';
	import * as Command from '$lib/components/ui/command/index.js';
	import * as Dialog from '$lib/components/ui/dialog/index.js';
	import * as Sheet from '$lib/components/ui/sheet/index.js';
	import * as Table from '$lib/components/ui/table/index.js';
	import * as Accordion from '$lib/components/ui/accordion';
	import { toast } from 'svelte-sonner';

	let items: any[] = [];
	const pageSize = 25;
	let total = 0;
	let next: string | null = null;
	let prev: string | null = null;
	let cursor: string | null = null;
	let cursorHistory: string[] = [];

	let autocomplete_institute: any[] = [];
	const cc = 'ID'; // Example country code

	let autocomplete_works: any[] = [];
	const ror = '04w4pwd42'; // Research Organization Registry ID of President University (https://ror.org/04w4pwd42)

	let autocomplete_authors: any[] = [];
	const id = 'i190243414'; // OpenAlex institution ID of President University (https://openalex.org/institutions/i190243414)

	let autocomplete_sources: any[] = [];

	let list_published: any[] = [];
	let list_statuses: any[] = [];
	let list_types: any[] = [];

	let searchTerm = '';
	let workId = '';
	let workTitle = '';
	let authorId = '';
	let authorName = '';
	let sourceId = '';
	let sourceName = '';
	let pubYear = '';
	let workType = '';
	let statusName = '';
	let institutionId = '';
	let instituteName = '';
	let fulltext = '';

	let citedBy = '';
	let cites = '';
	let related = '';

	const getInstitute = async () => {
		try {
			let api = `https://api.openalex.org/autocomplete/institutions?q=${instituteName}`;

			console.log(api);
			const response = await axios.get(api);
			autocomplete_institute = response.data.results;
			console.log('getInstitute successful', autocomplete_institute);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getInstitute:', error.message);
		}
	};

	// onMount(getInstitute);

	const getAuthors = async () => {
		try {
			let api = `https://api.openalex.org/autocomplete/authors?q=${authorName}`;

			console.log(api);
			const response = await axios.get(api);
			autocomplete_authors = response.data.results;
			console.log('getAuthors successful', autocomplete_authors);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getAuthors:', error.message);
		}
	};

	// onMount(getAuthors);

	const getWorks = async () => {
		try {
			let api = `https://api.openalex.org/autocomplete/works?q=${workTitle}&filter=publication_year:<${nxtYr}`;

			if (authorId.trim() !== '') {
				api += `,authorships.author.id:${encodeURIComponent(authorId.trim())}`;
			}

			if (sourceId.trim() !== '') {
				api += `,primary_location.source.id:${encodeURIComponent(sourceId.trim())}`;
			}

			if (pubYear !== '') {
				api += `,publication_year:${encodeURIComponent(pubYear)}`;
			}

			if (workType !== '') {
				api += `,type:${encodeURIComponent(workType)}`;
			}

			if (statusName !== '') {
				api += `,oa_status:${encodeURIComponent(statusName)}`;
			}

			if (workId !== '') {
				api += `,ids.openalex:${encodeURIComponent(workId)}`;
			}

			if (institutionId !== '') {
				api += `,institutions.id:${encodeURIComponent(institutionId)}`;
			}

			if (citedBy !== '') {
				api += `,cited_by:${encodeURIComponent(citedBy)}`;
			}

			if (cites !== '') {
				api += `,cites:${encodeURIComponent(cites)}`;
			}

			if (related !== '') {
				api += `,related_to:${encodeURIComponent(related)}`;
			}

			if (fulltext !== '') {
				api += `,has_fulltext:${fulltext}`;
			}

			console.log(api);
			const response = await axios.get(api);
			autocomplete_works = response.data.results;
			console.log('getWorks successful', autocomplete_works);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getWorks:', error.message);
		}
	};

	// onMount(getWorks);

	const getSources = async () => {
		try {
			let api = `https://api.openalex.org/autocomplete/sources?q=${sourceName}`;

			console.log(api);
			const response = await axios.get(api);
			autocomplete_sources = response.data.results;
			console.log('getSources successful', autocomplete_sources);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getSources:', error.message);
		}
	};

	// onMount(getSources);

	const getPublished = async () => {
		try {
			let api = `https://api.openalex.org/works?group_by=publication_year`;

			console.log(api);
			const response = await axios.get(api);
			list_published = response.data.group_by;
			console.log('getPublished successful', list_published);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getPublished:', error.message);
		}
	};

	// onMount(getPublished);

	const getTypes = async () => {
		try {
			let api = `https://api.openalex.org/works?group_by=type`;

			console.log(api);
			const response = await axios.get(api);
			list_types = response.data.group_by;
			console.log('getTypes successful', list_types);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getTypes:', error.message);
		}
	};

	// onMount(getTypes);

	const getStatuses = async () => {
		try {
			let api = `https://api.openalex.org/works?group_by=oa_status`;

			console.log(api);
			const response = await axios.get(api);
			list_statuses = response.data.group_by;
			console.log('getStatuses successful', list_statuses);
			console.log(response.data);
		} catch (error: any) {
			console.error('There was a problem with getStatuses:', error.message);
		}
	};

	// onMount(getStatuses);

	let citations: any[] = [];

	const getCitations = async (string: string) => {
		try {
			let api = `https://api.citeas.org/product/` + string;

			console.log(api);
			const response = await axios.get(api);
			citations = response.data.citations;
			console.log('getCitations successful');
			console.log(citations[0].citation);
			navigator.clipboard.writeText(citations[0].citation);

			toast.success('Citation copied', {
				description: citations[0].citation
				// action: {
				// 	label: 'Undo',
				// 	onClick: () => console.info('Undo')
				// }
			});
		} catch (error: any) {
			console.error('There was a problem with getCitations:', error.message);
		}
	};

	let today = new Date();
	let yyyy = today.getFullYear();
	let nxtYr = 0;

	nxtYr = yyyy += 5;

	const fetchData = async () => {
		try {
			let api = `https://api.openalex.org/works?cursor=${cursor || '*'}&per-page=${pageSize}&search=${searchTerm}&filter=publication_year:<${nxtYr}`;

			if (authorId.trim() !== '') {
				api += `,authorships.author.id:${encodeURIComponent(authorId.trim())}`;
			}

			if (sourceId.trim() !== '') {
				api += `,primary_location.source.id:${encodeURIComponent(sourceId.trim())}`;
			}

			if (pubYear !== '') {
				api += `,publication_year:${encodeURIComponent(pubYear)}`;
			}

			if (workType !== '') {
				api += `,type:${encodeURIComponent(workType)}`;
			}

			if (statusName !== '') {
				api += `,oa_status:${encodeURIComponent(statusName)}`;
			}

			if (workId !== '') {
				api += `,ids.openalex:${encodeURIComponent(workId)}`;
			}

			if (institutionId !== '') {
				api += `,institutions.id:${encodeURIComponent(institutionId)}`;
			}

			if (citedBy !== '') {
				api += `,cited_by:${encodeURIComponent(citedBy)}`;
			}

			if (cites !== '') {
				api += `,cites:${encodeURIComponent(cites)}`;
			}

			if (related !== '') {
				api += `,related_to:${encodeURIComponent(related)}`;
			}

			if (fulltext !== '') {
				api += `,has_fulltext:${fulltext}`;
			}

			console.log(api);
			const response = await axios.get(api);
			items = response.data.results;
			console.log('fetchData successful', items);
			console.log(response.data);
			console.log(response.data.meta.count, 'results');

			total = response.data.meta.count;
			// Paging
			prev = cursorHistory.length ? cursorHistory[cursorHistory.length - 1] : '*';
			next = response.data.meta.next_cursor;
			console.log('next_cursor:', next);
			console.log('this_cursor:', cursor);
			console.log('prev_cursor:', prev);
		} catch (error: any) {
			console.error('There was a problem with the GET request:', error.message);
		}
	};

	onMount(fetchData);

	async function nextPage() {
		if (next) {
			cursorHistory.push(cursor || '*'); // Save current cursor to history
			cursor = next;
			await fetchData();
		}
	}

	async function prevPage() {
		if (cursorHistory.length > 0) {
			cursor = cursorHistory.pop() || '*'; // Restore previous cursor from history
			await fetchData();
		}
	}

	const autocompleteInstitute = async (event: any) => {
		instituteName = event.target.value;
		await getInstitute();
	};

	const autocompleteAuthor = async (event: any) => {
		authorName = event.target.value;
		await getAuthors();
	};

	const autocompleteWork = async (event: any) => {
		workTitle = event.target.value;
		await getWorks();
	};

	const autocompleteSource = async (event: any) => {
		sourceName = event.target.value;
		await getSources();
	};

	function runSearch(string: string) {
		cursor = '*';
		searchTerm = string;
		workId = '';
		workTitle = '';
		console.log(searchTerm);
		fetchData();
	}

	function clearSearch() {
		cursor = '*';
		searchTerm = '';
		workId = '';
		workTitle = '';
		authorId = '';
		authorName = '';
		sourceId = '';
		sourceName = '';
		pubYear = '';
		workType = '';
		statusName = '';
		institutionId = '';
		instituteName = '';
		citedBy = '';
		cites = '';
		related = '';
		fulltext = '';
		fetchData();
	}

	function clearInstitute() {
		cursor = '*';
		searchTerm = '';
		institutionId = '';
		instituteName = '';
		fetchData();
	}

	function clearWork() {
		cursor = '*';
		searchTerm = '';
		workId = '';
		workTitle = '';
		fetchData();
	}

	function clearAuthor() {
		cursor = '*';
		authorId = '';
		authorName = '';
		fetchData();
	}

	function clearSource() {
		cursor = '*';
		sourceId = '';
		sourceName = '';
		fetchData();
	}

	function clearPublished() {
		cursor = '*';
		pubYear = '';
		fetchData();
	}

	function clearType() {
		cursor = '*';
		workType = '';
		fetchData();
	}

	function clearStatus() {
		cursor = '*';
		statusName = '';
		fetchData();
	}

	function clearCites() {
		cursor = '*';
		cites = '';
		fetchData();
	}

	function clearCitedby() {
		cursor = '*';
		citedBy = '';
		fetchData();
	}

	function clearRelated() {
		cursor = '*';
		related = '';
		fetchData();
	}

	function setAuthor(id: string, string: string) {
		cursor = '*';
		authorId = id;
		authorName = string;
		console.log('[' + authorId + ']', authorName);
		fetchData();
	}

	function setWork(id: string, string: string) {
		cursor = '*';
		searchTerm = '';
		workId = id;
		workTitle = string;
		console.log('[' + workId + ']', workTitle);
		fetchData();
	}

	function setInstitute(id: string, string: string) {
		cursor = '*';
		institutionId = id;
		instituteName = string;
		console.log('[' + institutionId + ']', instituteName);
		fetchData();
	}

	function setSource(id: string, string: string) {
		cursor = '*';
		sourceId = id;
		sourceName = string;
		console.log('[' + sourceId + ']', sourceName);
		fetchData();
	}

	function setPubYear(string: string) {
		cursor = '*';
		pubYear = string;
		console.log(pubYear);
		fetchData();
	}

	function setWorkType(string: string) {
		cursor = '*';
		workType = string;
		console.log(workType);
		fetchData();
	}

	function setStatus(string: string) {
		cursor = '*';
		statusName = string;
		console.log(statusName);
		fetchData();
	}

	function setCitedBy(id: string) {
		cursor = '*';
		searchTerm = '';
		workId = '';
		workTitle = '';
		citedBy = id;
		cites = '';
		related = '';
		console.log(citedBy);
		fetchData();
	}

	function setCites(id: string) {
		cursor = '*';
		searchTerm = '';
		workId = '';
		workTitle = '';
		citedBy = '';
		cites = id;
		related = '';
		console.log(cites);
		fetchData();
	}

	function setRelated(id: string) {
		cursor = '*';
		searchTerm = '';
		workId = '';
		workTitle = '';
		citedBy = '';
		cites = '';
		related = id;
		console.log(related);
		fetchData();
	}

	function has_fulltext() {
		cursor = '*';
		fulltext = 'true';
		fetchData();
	}

	function clear_fulltext() {
		cursor = '*';
		fulltext = '';
		fetchData();
	}

	type InvertedIndex = {
		[key: string]: number[];
	};

	const abstract_inverted_index: InvertedIndex = {};

	// Function to generate the paragraph
	function generateParagraph(index: InvertedIndex): string {
		// Create an array of word-position pairs
		const wordsWithPositions: [string, number[]][] = Object.entries(index);

		// Flatten and sort words by their positions
		const sortedWords: [string, number][] = wordsWithPositions
			.flatMap(([word, positions]) =>
				positions.map((position) => [word, position] as [string, number])
			)
			.sort((a, b) => a[1] - b[1]);

		// Construct the paragraph by concatenating words in sorted order
		const paragraph = sortedWords.map(([word]) => word).join(' ');

		return paragraph;
	}

	// Generate the paragraph
	const paragraph = generateParagraph(abstract_inverted_index);
	// console.log(paragraph);
</script>

<div id="markdown" class="markdown pb-12 pt-8">
	<div class="group relative my-4 flex flex-col space-y-2">
		<div class="relative mr-auto w-full">
			<div
				class="relative mt-2 rounded-md border ring-offset-background focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2"
			>
				<div class="theme-zinc w-full">
					<div class="preview flex min-h-[350px] w-full items-center justify-center p-10">
						<div class="relative w-full overflow-auto">
							<Accordion.Root>
								<Accordion.Item value="item-1">
									<Accordion.Trigger>
										{#if institutionId || authorId || pubYear || workType || statusName || sourceId || citedBy || cites || related || fulltext !== ''}
											Filters are applied
										{:else}
											<t class="text text-muted-foreground"> No filters applied </t>
										{/if}
									</Accordion.Trigger>
									<Accordion.Content>
										<div class="flex items-center pb-4">
											<Dialog.Root>
												<div class="w-full flex-1 md:w-auto md:flex-none">
													<Dialog.Trigger>
														<Button size="sm" variant="outline" on:click={autocompleteInstitute}>
															<svg
																xmlns="http://www.w3.org/2000/svg"
																width="16"
																height="16"
																viewBox="0 0 24 24"
																fill="none"
																stroke="currentColor"
																stroke-width="1.75"
																stroke-linecap="round"
																stroke-linejoin="round"
																class="lucide lucide-graduation-cap margin mr-2"
																><path
																	d="M21.42 10.922a1 1 0 0 0-.019-1.838L12.83 5.18a2 2 0 0 0-1.66 0L2.6 9.08a1 1 0 0 0 0 1.832l8.57 3.908a2 2 0 0 0 1.66 0z"
																/><path d="M22 10v6" /><path
																	d="M6 12.5V16a6 3 0 0 0 12 0v-3.5"
																/></svg
															>Institution
														</Button>
													</Dialog.Trigger>
													<Dialog.Content class="padding p-0">
														<Command.Root>
															<div class="flex items-center border-b px-3">
																<svg
																	class="mr-2 h-4 w-4 shrink-0 opacity-50"
																	viewBox="0 0 15 15"
																	fill="none"
																	xmlns="http://www.w3.org/2000/svg"
																	><path
																		d="M10 6.5C10 8.433 8.433 10 6.5 10C4.567 10 3 8.433 3 6.5C3 4.567 4.567 3 6.5 3C8.433 3 10 4.567 10 6.5ZM9.30884 10.0159C8.53901 10.6318 7.56251 11 6.5 11C4.01472 11 2 8.98528 2 6.5C2 4.01472 4.01472 2 6.5 2C8.98528 2 11 4.01472 11 6.5C11 7.56251 10.6318 8.53901 10.0159 9.30884L12.8536 12.1464C13.0488 12.3417 13.0488 12.6583 12.8536 12.8536C12.6583 13.0488 12.3417 13.0488 12.1464 12.8536L9.30884 10.0159Z"
																		fill="currentColor"
																		fill-rule="evenodd"
																		clip-rule="evenodd"
																	></path></svg
																>
																<input
																	class="flex h-11 w-full rounded-md bg-transparent py-3 text-sm outline-none placeholder:text-muted-foreground disabled:cursor-not-allowed disabled:opacity-50"
																	type="text"
																	bind:value={instituteName}
																	on:input={autocompleteInstitute}
																	placeholder="Type institution name"
																/>
															</div>
															<div
																class="
										padding relative flex cursor-default select-none items-center
										border-b p-2 text-sm outline-none aria-selected:bg-accent
										aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
															>
																<p
																	style="display: inline-flex; align-items:center"
																	class="text-sm text-orange-700 dark:text-orange-400"
																>
																	<svg
																		class="margin-right mr-2"
																		width="15"
																		height="15"
																		viewBox="0 0 15 15"
																		fill="none"
																		xmlns="http://www.w3.org/2000/svg"
																		><path
																			d="M7.49991 0.876892C3.84222 0.876892 0.877075 3.84204 0.877075 7.49972C0.877075 11.1574 3.84222 14.1226 7.49991 14.1226C11.1576 14.1226 14.1227 11.1574 14.1227 7.49972C14.1227 3.84204 11.1576 0.876892 7.49991 0.876892ZM1.82707 7.49972C1.82707 4.36671 4.36689 1.82689 7.49991 1.82689C10.6329 1.82689 13.1727 4.36671 13.1727 7.49972C13.1727 10.6327 10.6329 13.1726 7.49991 13.1726C4.36689 13.1726 1.82707 10.6327 1.82707 7.49972ZM8.24992 4.49999C8.24992 4.9142 7.91413 5.24999 7.49992 5.24999C7.08571 5.24999 6.74992 4.9142 6.74992 4.49999C6.74992 4.08577 7.08571 3.74999 7.49992 3.74999C7.91413 3.74999 8.24992 4.08577 8.24992 4.49999ZM6.00003 5.99999H6.50003H7.50003C7.77618 5.99999 8.00003 6.22384 8.00003 6.49999V9.99999H8.50003H9.00003V11H8.50003H7.50003H6.50003H6.00003V9.99999H6.50003H7.00003V6.99999H6.50003H6.00003V5.99999Z"
																			fill="currentColor"
																			fill-rule="evenodd"
																			clip-rule="evenodd"
																		></path></svg
																	>
																	Works count assumes other filters are inactive
																</p>
															</div>

															{#if institutionId !== ''}
																<Command.Group heading="Actions">
																	<Command.Item>
																		<button
																			class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																			on:click={clearInstitute}
																		>
																			<p style="display: inline-flex; align-items:center">
																				<svg
																					class="margin-right mr-2"
																					width="15"
																					height="15"
																					viewBox="0 0 15 15"
																					fill="none"
																					xmlns="http://www.w3.org/2000/svg"
																					><path
																						d="M4.85355 2.14645C5.04882 2.34171 5.04882 2.65829 4.85355 2.85355L3.70711 4H9C11.4853 4 13.5 6.01472 13.5 8.5C13.5 10.9853 11.4853 13 9 13H5C4.72386 13 4.5 12.7761 4.5 12.5C4.5 12.2239 4.72386 12 5 12H9C10.933 12 12.5 10.433 12.5 8.5C12.5 6.567 10.933 5 9 5H3.70711L4.85355 6.14645C5.04882 6.34171 5.04882 6.65829 4.85355 6.85355C4.65829 7.04882 4.34171 7.04882 4.14645 6.85355L2.14645 4.85355C1.95118 4.65829 1.95118 4.34171 2.14645 4.14645L4.14645 2.14645C4.34171 1.95118 4.65829 1.95118 4.85355 2.14645Z"
																						fill="currentColor"
																						fill-rule="evenodd"
																						clip-rule="evenodd"
																					></path></svg
																				>
																				Reset
																			</p>
																		</button>
																	</Command.Item>
																</Command.Group>
															{:else}
																<div></div>
															{/if}
															<Command.List>
																<Command.Group heading="Institutions">
																	<Command.Empty>No results found.</Command.Empty>
																	{#each autocomplete_institute as institute}
																		<Command.Item>
																			{#if institutionId == institute.id.substring('https://openalex.org/'.length)}
																				<button
																					class="text padding p-.5 text justify cursor-default select-none justify-items-start rounded-sm text-left text-sm text-muted-foreground outline-none aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																					on:click={clearInstitute}
																				>
																					{institute.display_name} <br /><span class="text text-xs"
																						>{institute.hint} · {institute.works_count} works
																					</span>
																				</button>
																			{:else}
																				<button
																					class="padding p-.5 text justify cursor-default select-none justify-items-start rounded-sm text-left text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																					on:click={() =>
																						setInstitute(
																							institute.id.substring(
																								'https://openalex.org/'.length
																							),
																							institute.display_name
																						)}
																				>
																					{institute.display_name} <br /><span class="text text-xs"
																						>{institute.hint} · {institute.works_count} works
																					</span>
																				</button>
																			{/if}
																		</Command.Item>
																	{/each}
																</Command.Group>
																<Command.Separator />
															</Command.List>
															{#if institutionId !== ''}
																<div
																	class="
													padding relative flex cursor-default select-none items-center
													rounded-sm p-2 text-sm outline-none hover:text-blue-500
													aria-selected:bg-accent aria-selected:text-accent-foreground
													data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																>
																	<p
																		style="display: inline-flex; align-items:center"
																		class="text-sm dark:text-lime-400"
																	>
																		<svg
																			class="margin-right mr-2"
																			width="15"
																			height="15"
																			viewBox="0 0 15 15"
																			fill="none"
																			xmlns="http://www.w3.org/2000/svg"
																			><path
																				d="M7.49991 0.877045C3.84222 0.877045 0.877075 3.84219 0.877075 7.49988C0.877075 11.1575 3.84222 14.1227 7.49991 14.1227C11.1576 14.1227 14.1227 11.1575 14.1227 7.49988C14.1227 3.84219 11.1576 0.877045 7.49991 0.877045ZM1.82708 7.49988C1.82708 4.36686 4.36689 1.82704 7.49991 1.82704C10.6329 1.82704 13.1727 4.36686 13.1727 7.49988C13.1727 10.6329 10.6329 13.1727 7.49991 13.1727C4.36689 13.1727 1.82708 10.6329 1.82708 7.49988ZM10.1589 5.53774C10.3178 5.31191 10.2636 5.00001 10.0378 4.84109C9.81194 4.68217 9.50004 4.73642 9.34112 4.96225L6.51977 8.97154L5.35681 7.78706C5.16334 7.59002 4.84677 7.58711 4.64973 7.78058C4.45268 7.97404 4.44978 8.29061 4.64325 8.48765L6.22658 10.1003C6.33054 10.2062 6.47617 10.2604 6.62407 10.2483C6.77197 10.2363 6.90686 10.1591 6.99226 10.0377L10.1589 5.53774Z"
																				fill="currentColor"
																				fill-rule="evenodd"
																				clip-rule="evenodd"
																			></path></svg
																		>
																		Filter active: {instituteName}
																	</p>
																</div>
															{:else}
																<div></div>
															{/if}
														</Command.Root>
													</Dialog.Content>
												</div>
											</Dialog.Root>
											{#if institutionId !== ''}
												<Button
													size="sm"
													variant="ghost"
													class="text text-md font-normal text-muted-foreground"
													on:click={clearInstitute}
												>
													{instituteName}
													<svg
														class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
														viewBox="0 0 15 15"
														fill="none"
														xmlns="http://www.w3.org/2000/svg"
														><path
															d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
															fill="currentColor"
															fill-rule="evenodd"
															clip-rule="evenodd"
														></path></svg
													>
												</Button>
											{:else}
												<div></div>
											{/if}
										</div>
										<div class="flex items-center pb-4">
											<Dialog.Root>
												<div class="w-full flex-1 md:w-auto md:flex-none">
													<Dialog.Trigger>
														<Button size="sm" variant="outline" on:click={autocompleteAuthor}>
															<svg
																class="margin mr-2"
																width="15"
																height="15"
																viewBox="0 0 15 15"
																fill="none"
																xmlns="http://www.w3.org/2000/svg"
																><path
																	d="M7.5 0.875C5.49797 0.875 3.875 2.49797 3.875 4.5C3.875 6.15288 4.98124 7.54738 6.49373 7.98351C5.2997 8.12901 4.27557 8.55134 3.50407 9.31167C2.52216 10.2794 2.02502 11.72 2.02502 13.5999C2.02502 13.8623 2.23769 14.0749 2.50002 14.0749C2.76236 14.0749 2.97502 13.8623 2.97502 13.5999C2.97502 11.8799 3.42786 10.7206 4.17091 9.9883C4.91536 9.25463 6.02674 8.87499 7.49995 8.87499C8.97317 8.87499 10.0846 9.25463 10.8291 9.98831C11.5721 10.7206 12.025 11.8799 12.025 13.5999C12.025 13.8623 12.2376 14.0749 12.5 14.0749C12.7623 14.075 12.975 13.8623 12.975 13.6C12.975 11.72 12.4778 10.2794 11.4959 9.31166C10.7244 8.55135 9.70025 8.12903 8.50625 7.98352C10.0187 7.5474 11.125 6.15289 11.125 4.5C11.125 2.49797 9.50203 0.875 7.5 0.875ZM4.825 4.5C4.825 3.02264 6.02264 1.825 7.5 1.825C8.97736 1.825 10.175 3.02264 10.175 4.5C10.175 5.97736 8.97736 7.175 7.5 7.175C6.02264 7.175 4.825 5.97736 4.825 4.5Z"
																	fill="currentColor"
																	fill-rule="evenodd"
																	clip-rule="evenodd"
																></path></svg
															>Author
														</Button>
													</Dialog.Trigger>
													<Dialog.Content class="padding p-0">
														<Command.Root>
															<div class="flex items-center border-b px-3">
																<svg
																	class="mr-2 h-4 w-4 shrink-0 opacity-50"
																	viewBox="0 0 15 15"
																	fill="none"
																	xmlns="http://www.w3.org/2000/svg"
																	><path
																		d="M10 6.5C10 8.433 8.433 10 6.5 10C4.567 10 3 8.433 3 6.5C3 4.567 4.567 3 6.5 3C8.433 3 10 4.567 10 6.5ZM9.30884 10.0159C8.53901 10.6318 7.56251 11 6.5 11C4.01472 11 2 8.98528 2 6.5C2 4.01472 4.01472 2 6.5 2C8.98528 2 11 4.01472 11 6.5C11 7.56251 10.6318 8.53901 10.0159 9.30884L12.8536 12.1464C13.0488 12.3417 13.0488 12.6583 12.8536 12.8536C12.6583 13.0488 12.3417 13.0488 12.1464 12.8536L9.30884 10.0159Z"
																		fill="currentColor"
																		fill-rule="evenodd"
																		clip-rule="evenodd"
																	></path></svg
																>
																<input
																	class="flex h-11 w-full rounded-md bg-transparent py-3 text-sm outline-none placeholder:text-muted-foreground disabled:cursor-not-allowed disabled:opacity-50"
																	type="text"
																	bind:value={authorName}
																	on:input={autocompleteAuthor}
																	placeholder="Type author name"
																/>
															</div>
															<div
																class="
									padding relative flex cursor-default select-none items-center
									border-b p-2 text-sm outline-none aria-selected:bg-accent
									aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
															>
																<p
																	style="display: inline-flex; align-items:center"
																	class="text-sm text-orange-700 dark:text-orange-400"
																>
																	<svg
																		class="margin-right mr-2"
																		width="15"
																		height="15"
																		viewBox="0 0 15 15"
																		fill="none"
																		xmlns="http://www.w3.org/2000/svg"
																		><path
																			d="M7.49991 0.876892C3.84222 0.876892 0.877075 3.84204 0.877075 7.49972C0.877075 11.1574 3.84222 14.1226 7.49991 14.1226C11.1576 14.1226 14.1227 11.1574 14.1227 7.49972C14.1227 3.84204 11.1576 0.876892 7.49991 0.876892ZM1.82707 7.49972C1.82707 4.36671 4.36689 1.82689 7.49991 1.82689C10.6329 1.82689 13.1727 4.36671 13.1727 7.49972C13.1727 10.6327 10.6329 13.1726 7.49991 13.1726C4.36689 13.1726 1.82707 10.6327 1.82707 7.49972ZM8.24992 4.49999C8.24992 4.9142 7.91413 5.24999 7.49992 5.24999C7.08571 5.24999 6.74992 4.9142 6.74992 4.49999C6.74992 4.08577 7.08571 3.74999 7.49992 3.74999C7.91413 3.74999 8.24992 4.08577 8.24992 4.49999ZM6.00003 5.99999H6.50003H7.50003C7.77618 5.99999 8.00003 6.22384 8.00003 6.49999V9.99999H8.50003H9.00003V11H8.50003H7.50003H6.50003H6.00003V9.99999H6.50003H7.00003V6.99999H6.50003H6.00003V5.99999Z"
																			fill="currentColor"
																			fill-rule="evenodd"
																			clip-rule="evenodd"
																		></path></svg
																	>
																	Works count assumes other filters are inactive
																</p>
															</div>

															{#if authorId !== ''}
																<Command.Group heading="Actions">
																	<Command.Item>
																		<button
																			class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																			on:click={clearAuthor}
																		>
																			<p style="display: inline-flex; align-items:center">
																				<svg
																					class="margin-right mr-2"
																					width="15"
																					height="15"
																					viewBox="0 0 15 15"
																					fill="none"
																					xmlns="http://www.w3.org/2000/svg"
																					><path
																						d="M4.85355 2.14645C5.04882 2.34171 5.04882 2.65829 4.85355 2.85355L3.70711 4H9C11.4853 4 13.5 6.01472 13.5 8.5C13.5 10.9853 11.4853 13 9 13H5C4.72386 13 4.5 12.7761 4.5 12.5C4.5 12.2239 4.72386 12 5 12H9C10.933 12 12.5 10.433 12.5 8.5C12.5 6.567 10.933 5 9 5H3.70711L4.85355 6.14645C5.04882 6.34171 5.04882 6.65829 4.85355 6.85355C4.65829 7.04882 4.34171 7.04882 4.14645 6.85355L2.14645 4.85355C1.95118 4.65829 1.95118 4.34171 2.14645 4.14645L4.14645 2.14645C4.34171 1.95118 4.65829 1.95118 4.85355 2.14645Z"
																						fill="currentColor"
																						fill-rule="evenodd"
																						clip-rule="evenodd"
																					></path></svg
																				>
																				Reset
																			</p>
																		</button>
																	</Command.Item>
																</Command.Group>
															{:else}
																<div></div>
															{/if}
															<Command.List>
																<Command.Group heading="Authors">
																	<Command.Empty>No results found.</Command.Empty>
																	{#each autocomplete_authors as author}
																		<Command.Item>
																			<!-- class="padding p-.5 relative flex cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50" -->
																			{#if authorId == author.id.substring('https://openalex.org/'.length)}
																				<button
																					class="text padding p-.5 text justify cursor-default select-none justify-items-start rounded-sm text-left text-sm text-muted-foreground outline-none aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																					on:click={clearAuthor}
																				>
																					{author.display_name} <br />
																					<span class="text text-xs">
																						{author.hint} · {author.works_count} works
																					</span>
																				</button>
																			{:else}
																				<button
																					class="padding p-.5 text justify cursor-default select-none justify-items-start rounded-sm text-left text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																					on:click={() =>
																						setAuthor(
																							author.id.substring('https://openalex.org/'.length),
																							author.display_name
																						)}
																				>
																					{author.display_name} <br />
																					<span class="text text-xs">
																						{author.hint} · {author.works_count} works
																					</span>
																				</button>
																			{/if}
																		</Command.Item>
																	{/each}
																</Command.Group>
																<Command.Separator />
															</Command.List>
															{#if authorId !== ''}
																<div
																	class="
													padding relative flex cursor-default select-none items-center
													rounded-sm p-2 text-sm outline-none hover:text-blue-500
													aria-selected:bg-accent aria-selected:text-accent-foreground
													data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																>
																	<p
																		style="display: inline-flex; align-items:center"
																		class="text-sm dark:text-lime-400"
																	>
																		<svg
																			class="margin-right mr-2"
																			width="15"
																			height="15"
																			viewBox="0 0 15 15"
																			fill="none"
																			xmlns="http://www.w3.org/2000/svg"
																			><path
																				d="M7.49991 0.877045C3.84222 0.877045 0.877075 3.84219 0.877075 7.49988C0.877075 11.1575 3.84222 14.1227 7.49991 14.1227C11.1576 14.1227 14.1227 11.1575 14.1227 7.49988C14.1227 3.84219 11.1576 0.877045 7.49991 0.877045ZM1.82708 7.49988C1.82708 4.36686 4.36689 1.82704 7.49991 1.82704C10.6329 1.82704 13.1727 4.36686 13.1727 7.49988C13.1727 10.6329 10.6329 13.1727 7.49991 13.1727C4.36689 13.1727 1.82708 10.6329 1.82708 7.49988ZM10.1589 5.53774C10.3178 5.31191 10.2636 5.00001 10.0378 4.84109C9.81194 4.68217 9.50004 4.73642 9.34112 4.96225L6.51977 8.97154L5.35681 7.78706C5.16334 7.59002 4.84677 7.58711 4.64973 7.78058C4.45268 7.97404 4.44978 8.29061 4.64325 8.48765L6.22658 10.1003C6.33054 10.2062 6.47617 10.2604 6.62407 10.2483C6.77197 10.2363 6.90686 10.1591 6.99226 10.0377L10.1589 5.53774Z"
																				fill="currentColor"
																				fill-rule="evenodd"
																				clip-rule="evenodd"
																			></path></svg
																		>
																		Filter active: {authorName}
																	</p>
																</div>
															{:else}
																<div></div>
															{/if}
														</Command.Root>
													</Dialog.Content>
												</div>
											</Dialog.Root>
											{#if authorId !== ''}
												<Button
													size="sm"
													variant="ghost"
													class="text text-md font-normal text-muted-foreground"
													on:click={clearAuthor}
												>
													{authorName}
													<svg
														class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
														viewBox="0 0 15 15"
														fill="none"
														xmlns="http://www.w3.org/2000/svg"
														><path
															d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
															fill="currentColor"
															fill-rule="evenodd"
															clip-rule="evenodd"
														></path></svg
													>
												</Button>
											{:else}
												<div></div>
											{/if}
										</div>
										<div class="flex items-center pb-4">
											<Dialog.Root>
												<div class="w-full flex-1 md:w-auto md:flex-none">
													<Dialog.Trigger>
														<Button size="sm" variant="outline" on:click={getPublished}>
															<svg
																class="margin mr-2"
																width="15"
																height="15"
																viewBox="0 0 15 15"
																fill="none"
																xmlns="http://www.w3.org/2000/svg"
																><path
																	d="M4.5 1C4.77614 1 5 1.22386 5 1.5V2H10V1.5C10 1.22386 10.2239 1 10.5 1C10.7761 1 11 1.22386 11 1.5V2H12.5C13.3284 2 14 2.67157 14 3.5V12.5C14 13.3284 13.3284 14 12.5 14H2.5C1.67157 14 1 13.3284 1 12.5V3.5C1 2.67157 1.67157 2 2.5 2H4V1.5C4 1.22386 4.22386 1 4.5 1ZM10 3V3.5C10 3.77614 10.2239 4 10.5 4C10.7761 4 11 3.77614 11 3.5V3H12.5C12.7761 3 13 3.22386 13 3.5V5H2V3.5C2 3.22386 2.22386 3 2.5 3H4V3.5C4 3.77614 4.22386 4 4.5 4C4.77614 4 5 3.77614 5 3.5V3H10ZM2 6V12.5C2 12.7761 2.22386 13 2.5 13H12.5C12.7761 13 13 12.7761 13 12.5V6H2ZM7 7.5C7 7.22386 7.22386 7 7.5 7C7.77614 7 8 7.22386 8 7.5C8 7.77614 7.77614 8 7.5 8C7.22386 8 7 7.77614 7 7.5ZM9.5 7C9.22386 7 9 7.22386 9 7.5C9 7.77614 9.22386 8 9.5 8C9.77614 8 10 7.77614 10 7.5C10 7.22386 9.77614 7 9.5 7ZM11 7.5C11 7.22386 11.2239 7 11.5 7C11.7761 7 12 7.22386 12 7.5C12 7.77614 11.7761 8 11.5 8C11.2239 8 11 7.77614 11 7.5ZM11.5 9C11.2239 9 11 9.22386 11 9.5C11 9.77614 11.2239 10 11.5 10C11.7761 10 12 9.77614 12 9.5C12 9.22386 11.7761 9 11.5 9ZM9 9.5C9 9.22386 9.22386 9 9.5 9C9.77614 9 10 9.22386 10 9.5C10 9.77614 9.77614 10 9.5 10C9.22386 10 9 9.77614 9 9.5ZM7.5 9C7.22386 9 7 9.22386 7 9.5C7 9.77614 7.22386 10 7.5 10C7.77614 10 8 9.77614 8 9.5C8 9.22386 7.77614 9 7.5 9ZM5 9.5C5 9.22386 5.22386 9 5.5 9C5.77614 9 6 9.22386 6 9.5C6 9.77614 5.77614 10 5.5 10C5.22386 10 5 9.77614 5 9.5ZM3.5 9C3.22386 9 3 9.22386 3 9.5C3 9.77614 3.22386 10 3.5 10C3.77614 10 4 9.77614 4 9.5C4 9.22386 3.77614 9 3.5 9ZM3 11.5C3 11.2239 3.22386 11 3.5 11C3.77614 11 4 11.2239 4 11.5C4 11.7761 3.77614 12 3.5 12C3.22386 12 3 11.7761 3 11.5ZM5.5 11C5.22386 11 5 11.2239 5 11.5C5 11.7761 5.22386 12 5.5 12C5.77614 12 6 11.7761 6 11.5C6 11.2239 5.77614 11 5.5 11ZM7 11.5C7 11.2239 7.22386 11 7.5 11C7.77614 11 8 11.2239 8 11.5C8 11.7761 7.77614 12 7.5 12C7.22386 12 7 11.7761 7 11.5ZM9.5 11C9.22386 11 9 11.2239 9 11.5C9 11.7761 9.22386 12 9.5 12C9.77614 12 10 11.7761 10 11.5C10 11.2239 9.77614 11 9.5 11Z"
																	fill="currentColor"
																	fill-rule="evenodd"
																	clip-rule="evenodd"
																></path></svg
															>Year
														</Button>
													</Dialog.Trigger>
													<Dialog.Content class="padding p-0">
														<Command.Root>
															<Command.Input placeholder="Type publication year" />
															<div
																class="
											padding relative flex cursor-default select-none items-center
											rounded-sm p-2 text-sm outline-none hover:text-blue-500
											aria-selected:bg-accent aria-selected:text-accent-foreground
											data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
															>
																<p
																	style="display: inline-flex; align-items:center"
																	class="text-sm text-orange-700 dark:text-orange-400"
																>
																	<svg
																		class="margin-right mr-2"
																		width="15"
																		height="15"
																		viewBox="0 0 15 15"
																		fill="none"
																		xmlns="http://www.w3.org/2000/svg"
																		><path
																			d="M7.49991 0.876892C3.84222 0.876892 0.877075 3.84204 0.877075 7.49972C0.877075 11.1574 3.84222 14.1226 7.49991 14.1226C11.1576 14.1226 14.1227 11.1574 14.1227 7.49972C14.1227 3.84204 11.1576 0.876892 7.49991 0.876892ZM1.82707 7.49972C1.82707 4.36671 4.36689 1.82689 7.49991 1.82689C10.6329 1.82689 13.1727 4.36671 13.1727 7.49972C13.1727 10.6327 10.6329 13.1726 7.49991 13.1726C4.36689 13.1726 1.82707 10.6327 1.82707 7.49972ZM8.24992 4.49999C8.24992 4.9142 7.91413 5.24999 7.49992 5.24999C7.08571 5.24999 6.74992 4.9142 6.74992 4.49999C6.74992 4.08577 7.08571 3.74999 7.49992 3.74999C7.91413 3.74999 8.24992 4.08577 8.24992 4.49999ZM6.00003 5.99999H6.50003H7.50003C7.77618 5.99999 8.00003 6.22384 8.00003 6.49999V9.99999H8.50003H9.00003V11H8.50003H7.50003H6.50003H6.00003V9.99999H6.50003H7.00003V6.99999H6.50003H6.00003V5.99999Z"
																			fill="currentColor"
																			fill-rule="evenodd"
																			clip-rule="evenodd"
																		></path></svg
																	>
																	Works count assumes other filters are inactive
																</p>
															</div>
															<Command.Separator />

															{#if pubYear !== ''}
																<Command.Group heading="Actions">
																	<Command.Item>
																		<button
																			class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																			on:click={clearPublished}
																		>
																			<p style="display: inline-flex; align-items:center">
																				<svg
																					class="margin-right mr-2"
																					width="15"
																					height="15"
																					viewBox="0 0 15 15"
																					fill="none"
																					xmlns="http://www.w3.org/2000/svg"
																					><path
																						d="M4.85355 2.14645C5.04882 2.34171 5.04882 2.65829 4.85355 2.85355L3.70711 4H9C11.4853 4 13.5 6.01472 13.5 8.5C13.5 10.9853 11.4853 13 9 13H5C4.72386 13 4.5 12.7761 4.5 12.5C4.5 12.2239 4.72386 12 5 12H9C10.933 12 12.5 10.433 12.5 8.5C12.5 6.567 10.933 5 9 5H3.70711L4.85355 6.14645C5.04882 6.34171 5.04882 6.65829 4.85355 6.85355C4.65829 7.04882 4.34171 7.04882 4.14645 6.85355L2.14645 4.85355C1.95118 4.65829 1.95118 4.34171 2.14645 4.14645L4.14645 2.14645C4.34171 1.95118 4.65829 1.95118 4.85355 2.14645Z"
																						fill="currentColor"
																						fill-rule="evenodd"
																						clip-rule="evenodd"
																					></path></svg
																				>
																				Reset
																			</p>
																		</button>
																	</Command.Item>
																</Command.Group>
															{:else}
																<div></div>
															{/if}
															<Command.List>
																<Command.Group heading="Publication Year">
																	<Command.Empty>No results found.</Command.Empty>
																	{#each list_published as published}
																		<!-- {#if published.count < 1 || published.key_display_name > yyyy} -->
																		{#if published.count < 1}
																			<div></div>
																		{:else}
																			<Command.Item>
																				<!-- class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50" -->
																				<button
																					class="padding p-.5 text justify cursor-default select-none justify-items-start rounded-sm text-left text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																					on:click={() => setPubYear(published.key_display_name)}
																				>
																					<!-- <t class="text overflow-auto whitespace-normal text-left">
																						{published.key_display_name}
																						({published.count})
																						</t> -->
																					{published.key_display_name} <br />
																					<span class="text text-xs">{published.count} works</span>
																				</button>
																			</Command.Item>
																		{/if}
																	{/each}
																</Command.Group>
																<Command.Separator />
															</Command.List>
															{#if pubYear !== ''}
																<div
																	class="
														padding relative flex cursor-default select-none items-center
														rounded-sm p-2 text-sm outline-none hover:text-blue-500
														aria-selected:bg-accent aria-selected:text-accent-foreground
														data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																>
																	<p
																		style="display: inline-flex; align-items:center"
																		class="text-sm dark:text-lime-400"
																	>
																		<svg
																			class="margin-right mr-2"
																			width="15"
																			height="15"
																			viewBox="0 0 15 15"
																			fill="none"
																			xmlns="http://www.w3.org/2000/svg"
																			><path
																				d="M7.49991 0.877045C3.84222 0.877045 0.877075 3.84219 0.877075 7.49988C0.877075 11.1575 3.84222 14.1227 7.49991 14.1227C11.1576 14.1227 14.1227 11.1575 14.1227 7.49988C14.1227 3.84219 11.1576 0.877045 7.49991 0.877045ZM1.82708 7.49988C1.82708 4.36686 4.36689 1.82704 7.49991 1.82704C10.6329 1.82704 13.1727 4.36686 13.1727 7.49988C13.1727 10.6329 10.6329 13.1727 7.49991 13.1727C4.36689 13.1727 1.82708 10.6329 1.82708 7.49988ZM10.1589 5.53774C10.3178 5.31191 10.2636 5.00001 10.0378 4.84109C9.81194 4.68217 9.50004 4.73642 9.34112 4.96225L6.51977 8.97154L5.35681 7.78706C5.16334 7.59002 4.84677 7.58711 4.64973 7.78058C4.45268 7.97404 4.44978 8.29061 4.64325 8.48765L6.22658 10.1003C6.33054 10.2062 6.47617 10.2604 6.62407 10.2483C6.77197 10.2363 6.90686 10.1591 6.99226 10.0377L10.1589 5.53774Z"
																				fill="currentColor"
																				fill-rule="evenodd"
																				clip-rule="evenodd"
																			></path></svg
																		>
																		Filter active: {pubYear}
																	</p>
																</div>
															{:else}
																<div></div>
															{/if}
														</Command.Root>
													</Dialog.Content>
												</div>
											</Dialog.Root>
											{#if pubYear !== ''}
												<Button
													size="sm"
													variant="ghost"
													class="text text-md font-normal text-muted-foreground"
													on:click={clearPublished}
												>
													{pubYear}
													<svg
														class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
														viewBox="0 0 15 15"
														fill="none"
														xmlns="http://www.w3.org/2000/svg"
														><path
															d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
															fill="currentColor"
															fill-rule="evenodd"
															clip-rule="evenodd"
														></path></svg
													>
												</Button>
											{:else}
												<div></div>
											{/if}
										</div>
										<div class="flex items-center pb-4">
											<Dialog.Root>
												<div class="w-full flex-1 md:w-auto md:flex-none">
													<Dialog.Trigger>
														<Button size="sm" variant="outline" on:click={autocompleteSource}>
															<svg
																class="margin mr-2"
																width="15"
																height="15"
																viewBox="0 0 15 15"
																fill="none"
																xmlns="http://www.w3.org/2000/svg"
																><path
																	d="M4.62471 4.00001L4.56402 4.00001C4.04134 3.99993 3.70687 3.99988 3.4182 4.055C2.2379 4.28039 1.29846 5.17053 1.05815 6.33035C0.999538 6.61321 0.999604 6.93998 0.999703 7.43689L0.999711 7.50001L0.999703 7.56313C0.999604 8.06004 0.999538 8.38681 1.05815 8.66967C1.29846 9.8295 2.2379 10.7196 3.4182 10.945C3.70688 11.0001 4.04135 11.0001 4.56403 11L4.62471 11H5.49971C5.77585 11 5.99971 10.7762 5.99971 10.5C5.99971 10.2239 5.77585 10 5.49971 10H4.62471C4.02084 10 3.78907 9.99777 3.60577 9.96277C2.80262 9.8094 2.19157 9.21108 2.03735 8.46678C2.00233 8.29778 1.99971 8.08251 1.99971 7.50001C1.99971 6.91752 2.00233 6.70225 2.03735 6.53324C2.19157 5.78895 2.80262 5.19062 3.60577 5.03725C3.78907 5.00225 4.02084 5.00001 4.62471 5.00001H5.49971C5.77585 5.00001 5.99971 4.77615 5.99971 4.50001C5.99971 4.22387 5.77585 4.00001 5.49971 4.00001H4.62471ZM10.3747 5.00001C10.9786 5.00001 11.2104 5.00225 11.3937 5.03725C12.1968 5.19062 12.8079 5.78895 12.9621 6.53324C12.9971 6.70225 12.9997 6.91752 12.9997 7.50001C12.9997 8.08251 12.9971 8.29778 12.9621 8.46678C12.8079 9.21108 12.1968 9.8094 11.3937 9.96277C11.2104 9.99777 10.9786 10 10.3747 10H9.49971C9.22357 10 8.99971 10.2239 8.99971 10.5C8.99971 10.7762 9.22357 11 9.49971 11H10.3747L10.4354 11C10.9581 11.0001 11.2925 11.0001 11.5812 10.945C12.7615 10.7196 13.701 9.8295 13.9413 8.66967C13.9999 8.38681 13.9998 8.06005 13.9997 7.56314L13.9997 7.50001L13.9997 7.43688C13.9998 6.93998 13.9999 6.61321 13.9413 6.33035C13.701 5.17053 12.7615 4.28039 11.5812 4.055C11.2925 3.99988 10.9581 3.99993 10.4354 4.00001L10.3747 4.00001H9.49971C9.22357 4.00001 8.99971 4.22387 8.99971 4.50001C8.99971 4.77615 9.22357 5.00001 9.49971 5.00001H10.3747ZM5.00038 7C4.72424 7 4.50038 7.22386 4.50038 7.5C4.50038 7.77614 4.72424 8 5.00038 8H10.0004C10.2765 8 10.5004 7.77614 10.5004 7.5C10.5004 7.22386 10.2765 7 10.0004 7H5.00038Z"
																	fill="currentColor"
																	fill-rule="evenodd"
																	clip-rule="evenodd"
																></path></svg
															>Source
														</Button>
													</Dialog.Trigger>
													<Dialog.Content class="padding p-0">
														<Command.Root>
															<div class="flex items-center border-b px-3">
																<svg
																	class="mr-2 h-4 w-4 shrink-0 opacity-50"
																	viewBox="0 0 15 15"
																	fill="none"
																	xmlns="http://www.w3.org/2000/svg"
																	><path
																		d="M10 6.5C10 8.433 8.433 10 6.5 10C4.567 10 3 8.433 3 6.5C3 4.567 4.567 3 6.5 3C8.433 3 10 4.567 10 6.5ZM9.30884 10.0159C8.53901 10.6318 7.56251 11 6.5 11C4.01472 11 2 8.98528 2 6.5C2 4.01472 4.01472 2 6.5 2C8.98528 2 11 4.01472 11 6.5C11 7.56251 10.6318 8.53901 10.0159 9.30884L12.8536 12.1464C13.0488 12.3417 13.0488 12.6583 12.8536 12.8536C12.6583 13.0488 12.3417 13.0488 12.1464 12.8536L9.30884 10.0159Z"
																		fill="currentColor"
																		fill-rule="evenodd"
																		clip-rule="evenodd"
																	></path></svg
																>
																<input
																	class="flex h-11 w-full rounded-md bg-transparent py-3 text-sm outline-none placeholder:text-muted-foreground disabled:cursor-not-allowed disabled:opacity-50"
																	type="text"
																	bind:value={sourceName}
																	on:input={autocompleteSource}
																	placeholder="Type source name"
																/>
															</div>
															{#if sourceId !== ''}
																<Command.Group heading="Actions">
																	<Command.Item>
																		<button
																			class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																			on:click={clearSource}
																		>
																			<p style="display: inline-flex; align-items:center">
																				<svg
																					class="margin-right mr-2"
																					width="15"
																					height="15"
																					viewBox="0 0 15 15"
																					fill="none"
																					xmlns="http://www.w3.org/2000/svg"
																					><path
																						d="M4.85355 2.14645C5.04882 2.34171 5.04882 2.65829 4.85355 2.85355L3.70711 4H9C11.4853 4 13.5 6.01472 13.5 8.5C13.5 10.9853 11.4853 13 9 13H5C4.72386 13 4.5 12.7761 4.5 12.5C4.5 12.2239 4.72386 12 5 12H9C10.933 12 12.5 10.433 12.5 8.5C12.5 6.567 10.933 5 9 5H3.70711L4.85355 6.14645C5.04882 6.34171 5.04882 6.65829 4.85355 6.85355C4.65829 7.04882 4.34171 7.04882 4.14645 6.85355L2.14645 4.85355C1.95118 4.65829 1.95118 4.34171 2.14645 4.14645L4.14645 2.14645C4.34171 1.95118 4.65829 1.95118 4.85355 2.14645Z"
																						fill="currentColor"
																						fill-rule="evenodd"
																						clip-rule="evenodd"
																					></path></svg
																				>
																				Reset
																			</p>
																		</button>
																	</Command.Item>
																</Command.Group>
															{:else}
																<div></div>
															{/if}
															<Command.List>
																<Command.Group heading="Sources">
																	<Command.Empty>No results found.</Command.Empty>
																	{#each autocomplete_sources as source}
																		{#if sourceId == source.id.substring('https://openalex.org/'.length)}
																			<Command.Item>
																				<button
																					class="text padding p-.5 text justify cursor-default select-none justify-items-start rounded-sm text-left text-sm text-muted-foreground outline-none aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																					on:click={clearSource}
																				>
																					{source.display_name} <br />
																					<span class="text text-xs">{source.hint}</span>
																				</button>
																			</Command.Item>
																		{:else}
																			<Command.Item>
																				<button
																					class="padding p-.5 text justify cursor-default select-none justify-items-start rounded-sm text-left text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																					on:click={() =>
																						setSource(
																							source.id.substring('https://openalex.org/'.length),
																							source.display_name
																						)}
																				>
																					{source.display_name} <br />
																					<span class="text text-xs">{source.hint}</span>
																				</button>
																			</Command.Item>
																		{/if}
																	{/each}
																</Command.Group>
																<Command.Separator />
															</Command.List>
															{#if sourceId !== ''}
																<div
																	class="
																padding relative flex cursor-default select-none items-center
																rounded-sm p-2 text-sm outline-none hover:text-blue-500
																aria-selected:bg-accent aria-selected:text-accent-foreground
																data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																>
																	<p
																		style="display: inline-flex; align-items:center"
																		class="text-sm dark:text-lime-400"
																	>
																		<svg
																			class="margin-right mr-2"
																			width="15"
																			height="15"
																			viewBox="0 0 15 15"
																			fill="none"
																			xmlns="http://www.w3.org/2000/svg"
																			><path
																				d="M7.49991 0.877045C3.84222 0.877045 0.877075 3.84219 0.877075 7.49988C0.877075 11.1575 3.84222 14.1227 7.49991 14.1227C11.1576 14.1227 14.1227 11.1575 14.1227 7.49988C14.1227 3.84219 11.1576 0.877045 7.49991 0.877045ZM1.82708 7.49988C1.82708 4.36686 4.36689 1.82704 7.49991 1.82704C10.6329 1.82704 13.1727 4.36686 13.1727 7.49988C13.1727 10.6329 10.6329 13.1727 7.49991 13.1727C4.36689 13.1727 1.82708 10.6329 1.82708 7.49988ZM10.1589 5.53774C10.3178 5.31191 10.2636 5.00001 10.0378 4.84109C9.81194 4.68217 9.50004 4.73642 9.34112 4.96225L6.51977 8.97154L5.35681 7.78706C5.16334 7.59002 4.84677 7.58711 4.64973 7.78058C4.45268 7.97404 4.44978 8.29061 4.64325 8.48765L6.22658 10.1003C6.33054 10.2062 6.47617 10.2604 6.62407 10.2483C6.77197 10.2363 6.90686 10.1591 6.99226 10.0377L10.1589 5.53774Z"
																				fill="currentColor"
																				fill-rule="evenodd"
																				clip-rule="evenodd"
																			></path></svg
																		>
																		Filter active: openalex.org/{sourceId}
																	</p>
																</div>
															{:else}
																<div></div>
															{/if}
														</Command.Root>
													</Dialog.Content>
												</div>
											</Dialog.Root>
											{#if sourceId !== ''}
												<Button
													size="sm"
													variant="ghost"
													class="text text-md font-normal text-muted-foreground"
													on:click={clearSource}
												>
													{sourceName}
													<svg
														class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
														viewBox="0 0 15 15"
														fill="none"
														xmlns="http://www.w3.org/2000/svg"
														><path
															d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
															fill="currentColor"
															fill-rule="evenodd"
															clip-rule="evenodd"
														></path></svg
													>
												</Button>
											{:else}
												<div></div>
											{/if}
										</div>

										<div class="flex items-center pb-4">
											<Dialog.Root>
												<div class="w-full flex-1 md:w-auto md:flex-none">
													<Dialog.Trigger>
														<Button size="sm" variant="outline" on:click={getTypes}>
															<svg
																class="margin mr-2"
																width="15"
																height="15"
																viewBox="0 0 15 15"
																fill="none"
																xmlns="http://www.w3.org/2000/svg"
																><path
																	d="M3.5 2C3.22386 2 3 2.22386 3 2.5V12.5C3 12.7761 3.22386 13 3.5 13H11.5C11.7761 13 12 12.7761 12 12.5V6H8.5C8.22386 6 8 5.77614 8 5.5V2H3.5ZM9 2.70711L11.2929 5H9V2.70711ZM2 2.5C2 1.67157 2.67157 1 3.5 1H8.5C8.63261 1 8.75979 1.05268 8.85355 1.14645L12.8536 5.14645C12.9473 5.24021 13 5.36739 13 5.5V12.5C13 13.3284 12.3284 14 11.5 14H3.5C2.67157 14 2 13.3284 2 12.5V2.5Z"
																	fill="currentColor"
																	fill-rule="evenodd"
																	clip-rule="evenodd"
																></path></svg
															>Type
														</Button>
													</Dialog.Trigger>
													<Dialog.Content class="padding p-0">
														<Command.Root>
															<Command.Input placeholder="Search work type" />
															<div
																class="
												padding relative flex cursor-default select-none items-center
												rounded-sm p-2 text-sm outline-none hover:text-blue-500
												aria-selected:bg-accent aria-selected:text-accent-foreground
												data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
															>
																<p
																	style="display: inline-flex; align-items:center"
																	class="text-sm text-orange-700 dark:text-orange-400"
																>
																	<svg
																		class="margin-right mr-2"
																		width="15"
																		height="15"
																		viewBox="0 0 15 15"
																		fill="none"
																		xmlns="http://www.w3.org/2000/svg"
																		><path
																			d="M7.49991 0.876892C3.84222 0.876892 0.877075 3.84204 0.877075 7.49972C0.877075 11.1574 3.84222 14.1226 7.49991 14.1226C11.1576 14.1226 14.1227 11.1574 14.1227 7.49972C14.1227 3.84204 11.1576 0.876892 7.49991 0.876892ZM1.82707 7.49972C1.82707 4.36671 4.36689 1.82689 7.49991 1.82689C10.6329 1.82689 13.1727 4.36671 13.1727 7.49972C13.1727 10.6327 10.6329 13.1726 7.49991 13.1726C4.36689 13.1726 1.82707 10.6327 1.82707 7.49972ZM8.24992 4.49999C8.24992 4.9142 7.91413 5.24999 7.49992 5.24999C7.08571 5.24999 6.74992 4.9142 6.74992 4.49999C6.74992 4.08577 7.08571 3.74999 7.49992 3.74999C7.91413 3.74999 8.24992 4.08577 8.24992 4.49999ZM6.00003 5.99999H6.50003H7.50003C7.77618 5.99999 8.00003 6.22384 8.00003 6.49999V9.99999H8.50003H9.00003V11H8.50003H7.50003H6.50003H6.00003V9.99999H6.50003H7.00003V6.99999H6.50003H6.00003V5.99999Z"
																			fill="currentColor"
																			fill-rule="evenodd"
																			clip-rule="evenodd"
																		></path></svg
																	>
																	Work types count assumes other filters are inactive
																</p>
															</div>
															<Command.Separator />
															{#if workType !== ''}
																<Command.Group heading="Actions">
																	<Command.Item>
																		<button
																			class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																			on:click={clearType}
																		>
																			<p style="display: inline-flex; align-items:center">
																				<svg
																					class="margin-right mr-2"
																					width="15"
																					height="15"
																					viewBox="0 0 15 15"
																					fill="none"
																					xmlns="http://www.w3.org/2000/svg"
																					><path
																						d="M4.85355 2.14645C5.04882 2.34171 5.04882 2.65829 4.85355 2.85355L3.70711 4H9C11.4853 4 13.5 6.01472 13.5 8.5C13.5 10.9853 11.4853 13 9 13H5C4.72386 13 4.5 12.7761 4.5 12.5C4.5 12.2239 4.72386 12 5 12H9C10.933 12 12.5 10.433 12.5 8.5C12.5 6.567 10.933 5 9 5H3.70711L4.85355 6.14645C5.04882 6.34171 5.04882 6.65829 4.85355 6.85355C4.65829 7.04882 4.34171 7.04882 4.14645 6.85355L2.14645 4.85355C1.95118 4.65829 1.95118 4.34171 2.14645 4.14645L4.14645 2.14645C4.34171 1.95118 4.65829 1.95118 4.85355 2.14645Z"
																						fill="currentColor"
																						fill-rule="evenodd"
																						clip-rule="evenodd"
																					></path></svg
																				>
																				Reset
																			</p>
																		</button>
																	</Command.Item>
																</Command.Group>
															{:else}
																<div></div>
															{/if}
															<Command.List>
																<Command.Group heading="Work Types">
																	<Command.Empty>No results found.</Command.Empty>
																	{#each list_types as type}
																		{#if type.count < 1}
																			<div></div>
																		{:else}
																			<Command.Item>
																				<button
																					class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																					on:click={() => setWorkType(type.key_display_name)}
																				>
																					<!-- setWorkType(type.key_display_name)} -->
																					<t class="text overflow-auto whitespace-normal text-left">
																						{type.key_display_name} <br /><span class="text text-xs"
																							>{type.count} works
																						</span>
																					</t>
																				</button>
																			</Command.Item>
																		{/if}
																	{/each}
																</Command.Group>
																<Command.Separator />
															</Command.List>
															{#if workType !== ''}
																<div
																	class="
												padding relative flex cursor-default select-none items-center
												rounded-sm p-2 text-sm outline-none hover:text-blue-500
												aria-selected:bg-accent aria-selected:text-accent-foreground
												data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																>
																	<p
																		style="display: inline-flex; align-items:center"
																		class="text-sm dark:text-lime-400"
																	>
																		<svg
																			class="margin-right mr-2"
																			width="15"
																			height="15"
																			viewBox="0 0 15 15"
																			fill="none"
																			xmlns="http://www.w3.org/2000/svg"
																			><path
																				d="M7.49991 0.877045C3.84222 0.877045 0.877075 3.84219 0.877075 7.49988C0.877075 11.1575 3.84222 14.1227 7.49991 14.1227C11.1576 14.1227 14.1227 11.1575 14.1227 7.49988C14.1227 3.84219 11.1576 0.877045 7.49991 0.877045ZM1.82708 7.49988C1.82708 4.36686 4.36689 1.82704 7.49991 1.82704C10.6329 1.82704 13.1727 4.36686 13.1727 7.49988C13.1727 10.6329 10.6329 13.1727 7.49991 13.1727C4.36689 13.1727 1.82708 10.6329 1.82708 7.49988ZM10.1589 5.53774C10.3178 5.31191 10.2636 5.00001 10.0378 4.84109C9.81194 4.68217 9.50004 4.73642 9.34112 4.96225L6.51977 8.97154L5.35681 7.78706C5.16334 7.59002 4.84677 7.58711 4.64973 7.78058C4.45268 7.97404 4.44978 8.29061 4.64325 8.48765L6.22658 10.1003C6.33054 10.2062 6.47617 10.2604 6.62407 10.2483C6.77197 10.2363 6.90686 10.1591 6.99226 10.0377L10.1589 5.53774Z"
																				fill="currentColor"
																				fill-rule="evenodd"
																				clip-rule="evenodd"
																			></path></svg
																		>
																		Filter active: {workType}
																	</p>
																</div>
															{:else}
																<div></div>
															{/if}
														</Command.Root>
													</Dialog.Content>
												</div>
											</Dialog.Root>
											{#if workType !== ''}
												<Button
													size="sm"
													variant="ghost"
													class="text text-md font-normal text-muted-foreground"
													on:click={clearType}
												>
													{workType}
													<svg
														class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
														viewBox="0 0 15 15"
														fill="none"
														xmlns="http://www.w3.org/2000/svg"
														><path
															d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
															fill="currentColor"
															fill-rule="evenodd"
															clip-rule="evenodd"
														></path></svg
													>
												</Button>
											{:else}
												<div></div>
											{/if}
										</div>
										<div class="flex items-center pb-4">
											<Dialog.Root>
												<div class="w-full flex-1 md:w-auto md:flex-none">
													<Dialog.Trigger>
														<Button size="sm" variant="outline" on:click={getStatuses}>
															<svg
																class="margin mr-2"
																width="15"
																height="15"
																viewBox="0 0 15 15"
																fill="none"
																xmlns="http://www.w3.org/2000/svg"
																><path
																	d="M7.5 11C4.80285 11 2.52952 9.62184 1.09622 7.50001C2.52952 5.37816 4.80285 4 7.5 4C10.1971 4 12.4705 5.37816 13.9038 7.50001C12.4705 9.62183 10.1971 11 7.5 11ZM7.5 3C4.30786 3 1.65639 4.70638 0.0760002 7.23501C-0.0253338 7.39715 -0.0253334 7.60288 0.0760014 7.76501C1.65639 10.2936 4.30786 12 7.5 12C10.6921 12 13.3436 10.2936 14.924 7.76501C15.0253 7.60288 15.0253 7.39715 14.924 7.23501C13.3436 4.70638 10.6921 3 7.5 3ZM7.5 9.5C8.60457 9.5 9.5 8.60457 9.5 7.5C9.5 6.39543 8.60457 5.5 7.5 5.5C6.39543 5.5 5.5 6.39543 5.5 7.5C5.5 8.60457 6.39543 9.5 7.5 9.5Z"
																	fill="currentColor"
																	fill-rule="evenodd"
																	clip-rule="evenodd"
																></path></svg
															>Status
														</Button>
													</Dialog.Trigger>
													<Dialog.Content class="padding p-0">
														<Command.Root>
															<Command.Input placeholder="Type access status" />
															<div
																class="
												padding relative flex cursor-default select-none items-center
												rounded-sm p-2 text-sm outline-none hover:text-blue-500
												aria-selected:bg-accent aria-selected:text-accent-foreground
												data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
															>
																<p
																	style="display: inline-flex; align-items:center"
																	class="text-sm text-orange-700 dark:text-orange-400"
																>
																	<svg
																		class="margin-right mr-2"
																		width="15"
																		height="15"
																		viewBox="0 0 15 15"
																		fill="none"
																		xmlns="http://www.w3.org/2000/svg"
																		><path
																			d="M7.49991 0.876892C3.84222 0.876892 0.877075 3.84204 0.877075 7.49972C0.877075 11.1574 3.84222 14.1226 7.49991 14.1226C11.1576 14.1226 14.1227 11.1574 14.1227 7.49972C14.1227 3.84204 11.1576 0.876892 7.49991 0.876892ZM1.82707 7.49972C1.82707 4.36671 4.36689 1.82689 7.49991 1.82689C10.6329 1.82689 13.1727 4.36671 13.1727 7.49972C13.1727 10.6327 10.6329 13.1726 7.49991 13.1726C4.36689 13.1726 1.82707 10.6327 1.82707 7.49972ZM8.24992 4.49999C8.24992 4.9142 7.91413 5.24999 7.49992 5.24999C7.08571 5.24999 6.74992 4.9142 6.74992 4.49999C6.74992 4.08577 7.08571 3.74999 7.49992 3.74999C7.91413 3.74999 8.24992 4.08577 8.24992 4.49999ZM6.00003 5.99999H6.50003H7.50003C7.77618 5.99999 8.00003 6.22384 8.00003 6.49999V9.99999H8.50003H9.00003V11H8.50003H7.50003H6.50003H6.00003V9.99999H6.50003H7.00003V6.99999H6.50003H6.00003V5.99999Z"
																			fill="currentColor"
																			fill-rule="evenodd"
																			clip-rule="evenodd"
																		></path></svg
																	>
																	Access status count assumes other filters are inactive
																</p>
															</div>
															<Command.Separator />
															{#if statusName !== ''}
																<Command.Group heading="Actions">
																	<Command.Item>
																		<button
																			class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																			on:click={clearStatus}
																		>
																			<p style="display: inline-flex; align-items:center">
																				<svg
																					class="margin-right mr-2"
																					width="15"
																					height="15"
																					viewBox="0 0 15 15"
																					fill="none"
																					xmlns="http://www.w3.org/2000/svg"
																					><path
																						d="M4.85355 2.14645C5.04882 2.34171 5.04882 2.65829 4.85355 2.85355L3.70711 4H9C11.4853 4 13.5 6.01472 13.5 8.5C13.5 10.9853 11.4853 13 9 13H5C4.72386 13 4.5 12.7761 4.5 12.5C4.5 12.2239 4.72386 12 5 12H9C10.933 12 12.5 10.433 12.5 8.5C12.5 6.567 10.933 5 9 5H3.70711L4.85355 6.14645C5.04882 6.34171 5.04882 6.65829 4.85355 6.85355C4.65829 7.04882 4.34171 7.04882 4.14645 6.85355L2.14645 4.85355C1.95118 4.65829 1.95118 4.34171 2.14645 4.14645L4.14645 2.14645C4.34171 1.95118 4.65829 1.95118 4.85355 2.14645Z"
																						fill="currentColor"
																						fill-rule="evenodd"
																						clip-rule="evenodd"
																					></path></svg
																				>
																				Reset
																			</p>
																		</button>
																	</Command.Item>
																</Command.Group>
															{:else}
																<div></div>
															{/if}

															<Command.List>
																<Command.Group heading="Access Status">
																	<Command.Empty>No results found.</Command.Empty>
																	{#each list_statuses as status}
																		{#if status.count < 1}
																			<div></div>
																		{:else}
																			<Command.Item>
																				<button
																					class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																					on:click={() => setStatus(status.key_display_name)}
																				>
																					<t class="text overflow-auto whitespace-normal text-left">
																						{status.key_display_name} <br /><span
																							class="text text-xs"
																							>{status.count} works
																						</span>
																					</t>
																				</button>
																			</Command.Item>
																		{/if}
																	{/each}
																</Command.Group>
																<Command.Separator />
															</Command.List>
															{#if statusName !== ''}
																<div
																	class="
													padding relative flex cursor-default select-none items-center
													rounded-sm p-2 text-sm outline-none hover:text-blue-500
													aria-selected:bg-accent aria-selected:text-accent-foreground
													data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																>
																	<p
																		style="display: inline-flex; align-items:center"
																		class="text-sm dark:text-lime-400"
																	>
																		<svg
																			class="margin-right mr-2"
																			width="15"
																			height="15"
																			viewBox="0 0 15 15"
																			fill="none"
																			xmlns="http://www.w3.org/2000/svg"
																			><path
																				d="M7.49991 0.877045C3.84222 0.877045 0.877075 3.84219 0.877075 7.49988C0.877075 11.1575 3.84222 14.1227 7.49991 14.1227C11.1576 14.1227 14.1227 11.1575 14.1227 7.49988C14.1227 3.84219 11.1576 0.877045 7.49991 0.877045ZM1.82708 7.49988C1.82708 4.36686 4.36689 1.82704 7.49991 1.82704C10.6329 1.82704 13.1727 4.36686 13.1727 7.49988C13.1727 10.6329 10.6329 13.1727 7.49991 13.1727C4.36689 13.1727 1.82708 10.6329 1.82708 7.49988ZM10.1589 5.53774C10.3178 5.31191 10.2636 5.00001 10.0378 4.84109C9.81194 4.68217 9.50004 4.73642 9.34112 4.96225L6.51977 8.97154L5.35681 7.78706C5.16334 7.59002 4.84677 7.58711 4.64973 7.78058C4.45268 7.97404 4.44978 8.29061 4.64325 8.48765L6.22658 10.1003C6.33054 10.2062 6.47617 10.2604 6.62407 10.2483C6.77197 10.2363 6.90686 10.1591 6.99226 10.0377L10.1589 5.53774Z"
																				fill="currentColor"
																				fill-rule="evenodd"
																				clip-rule="evenodd"
																			></path></svg
																		>
																		Filter active: {statusName}
																	</p>
																</div>
															{:else}
																<div></div>
															{/if}
														</Command.Root>
													</Dialog.Content>
												</div>
											</Dialog.Root>
											{#if statusName !== ''}
												<Button
													size="sm"
													variant="ghost"
													class="text text-md font-normal text-muted-foreground"
													on:click={clearStatus}
												>
													{statusName}
													<svg
														class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
														viewBox="0 0 15 15"
														fill="none"
														xmlns="http://www.w3.org/2000/svg"
														><path
															d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
															fill="currentColor"
															fill-rule="evenodd"
															clip-rule="evenodd"
														></path></svg
													>
												</Button>
											{:else}
												<div></div>
											{/if}
										</div>
										<div class="items-center pb-4">
											<div class="display inline-flex">
												{#if cites !== ''}
													<Button
														size="sm"
														variant="outline"
														class="text text-md font-normal"
														on:click={clearCites}
													>
														Cites: {cites}
														<svg
															class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
															viewBox="0 0 15 15"
															fill="none"
															xmlns="http://www.w3.org/2000/svg"
															><path
																d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
																fill="currentColor"
																fill-rule="evenodd"
																clip-rule="evenodd"
															></path></svg
														>
													</Button>
												{:else}
													<Button
														size="sm"
														variant="ghost"
														class="text text-md font-normal"
														disabled>Cites</Button
													>
												{/if}
											</div>
										</div>
										<div class="items-center pb-4">
											<div class="display inline-flex">
												{#if citedBy !== ''}
													<Button
														size="sm"
														variant="outline"
														class="text text-md font-normal"
														on:click={clearCitedby}
													>
														Cited by: {citedBy}
														<svg
															class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
															viewBox="0 0 15 15"
															fill="none"
															xmlns="http://www.w3.org/2000/svg"
															><path
																d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
																fill="currentColor"
																fill-rule="evenodd"
																clip-rule="evenodd"
															></path></svg
														>
													</Button>
												{:else}
													<Button
														size="sm"
														variant="ghost"
														class="text text-md font-normal"
														disabled>Cited by</Button
													>
												{/if}
											</div>
										</div>
										<div class="items-center pb-0">
											<div class="display inline-flex">
												{#if related !== ''}
													<Button
														size="sm"
														variant="outline"
														class="text text-md font-normal"
														on:click={clearRelated}
													>
														Related to: {related}
														<svg
															class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
															viewBox="0 0 15 15"
															fill="none"
															xmlns="http://www.w3.org/2000/svg"
															><path
																d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
																fill="currentColor"
																fill-rule="evenodd"
																clip-rule="evenodd"
															></path></svg
														>
													</Button>
												{:else}
													<Button
														size="sm"
														variant="ghost"
														class="text text-md font-normal"
														disabled>Related to</Button
													>
												{/if}
											</div>
										</div>
									</Accordion.Content>
								</Accordion.Item>
							</Accordion.Root>

							<div class="flex items-center py-4">
								<div class="flex flex-1 items-center justify-between space-x-2 md:justify-start">
									<div class="w-full flex-1 md:w-auto md:flex-none">
										<form on:submit={() => runSearch(searchTerm)}>
											<!-- class="flex h-9 w-full max-w-sm rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 only:focus-visible:outline-none disabled:cursor-not-allowed disabled:opacity-50" -->
											<input
												class="flex h-9 w-screen max-w-sm rounded-md border border-input bg-background px-3 py-2 text-sm ring-offset-background file:border-0 file:bg-transparent file:text-sm file:font-medium placeholder:text-muted-foreground focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 only:focus-visible:outline-none disabled:cursor-not-allowed disabled:opacity-50"
												type="text"
												bind:value={searchTerm}
												placeholder="Title, abstract, fulltext"
											/>
										</form>
									</div>
									<Button variant="default" size="icon" on:click={() => runSearch(searchTerm)}>
										<svg
											xmlns="http://www.w3.org/2000/svg"
											viewBox="0 0 24 24"
											fill="none"
											stroke="currentColor"
											stroke-width="2"
											stroke-linecap="round"
											stroke-linejoin="round"
											class="lucide lucide-search h-[1.2rem] w-[1.2rem]"
											><circle cx="11" cy="11" r="8" /><path d="m21 21-4.3-4.3" /></svg
										>
									</Button>
									{#if workId !== '' || searchTerm == ''}
										<Button size="icon" style="visibility: hidden"></Button>
									{:else}
										<Button variant="ghost" size="icon" on:click={clearWork}>
											<svg
												width="15"
												height="15"
												viewBox="0 0 15 15"
												fill="none"
												xmlns="http://www.w3.org/2000/svg"
												><path
													d="M12.8536 2.85355C13.0488 2.65829 13.0488 2.34171 12.8536 2.14645C12.6583 1.95118 12.3417 1.95118 12.1464 2.14645L7.5 6.79289L2.85355 2.14645C2.65829 1.95118 2.34171 1.95118 2.14645 2.14645C1.95118 2.34171 1.95118 2.65829 2.14645 2.85355L6.79289 7.5L2.14645 12.1464C1.95118 12.3417 1.95118 12.6583 2.14645 12.8536C2.34171 13.0488 2.65829 13.0488 2.85355 12.8536L7.5 8.20711L12.1464 12.8536C12.3417 13.0488 12.6583 13.0488 12.8536 12.8536C13.0488 12.6583 13.0488 12.3417 12.8536 12.1464L8.20711 7.5L12.8536 2.85355Z"
													fill="currentColor"
													fill-rule="evenodd"
													clip-rule="evenodd"
												></path></svg
											>
										</Button>
									{/if}
								</div>
							</div>
							<div class="flex items-center py-4">
								<div class="flex flex-1 items-center justify-between space-x-2 md:justify-start">
									<Button variant="default" size="sm" on:click={clearSearch}>
										<svg
											class="margin-right mr-2"
											width="15"
											height="15"
											viewBox="0 0 15 15"
											fill="none"
											xmlns="http://www.w3.org/2000/svg"
											><path
												d="M4.85355 2.14645C5.04882 2.34171 5.04882 2.65829 4.85355 2.85355L3.70711 4H9C11.4853 4 13.5 6.01472 13.5 8.5C13.5 10.9853 11.4853 13 9 13H5C4.72386 13 4.5 12.7761 4.5 12.5C4.5 12.2239 4.72386 12 5 12H9C10.933 12 12.5 10.433 12.5 8.5C12.5 6.567 10.933 5 9 5H3.70711L4.85355 6.14645C5.04882 6.34171 5.04882 6.65829 4.85355 6.85355C4.65829 7.04882 4.34171 7.04882 4.14645 6.85355L2.14645 4.85355C1.95118 4.65829 1.95118 4.34171 2.14645 4.14645L4.14645 2.14645C4.34171 1.95118 4.65829 1.95118 4.85355 2.14645Z"
												fill="currentColor"
												fill-rule="evenodd"
												clip-rule="evenodd"
											></path></svg
										>RESET
									</Button>

									<Dialog.Root>
										<div class="w-full flex-1 md:w-auto md:flex-none">
											<Dialog.Trigger>
												{#if workId !== ''}
													<Button size="sm" variant="ghost" on:click={autocompleteWork}>
														<svg
															class="margin-right mr-2 dark:text-lime-400"
															width="15"
															height="15"
															viewBox="0 0 15 15"
															fill="none"
															xmlns="http://www.w3.org/2000/svg"
															><path
																d="M7.49991 0.877045C3.84222 0.877045 0.877075 3.84219 0.877075 7.49988C0.877075 11.1575 3.84222 14.1227 7.49991 14.1227C11.1576 14.1227 14.1227 11.1575 14.1227 7.49988C14.1227 3.84219 11.1576 0.877045 7.49991 0.877045ZM1.82708 7.49988C1.82708 4.36686 4.36689 1.82704 7.49991 1.82704C10.6329 1.82704 13.1727 4.36686 13.1727 7.49988C13.1727 10.6329 10.6329 13.1727 7.49991 13.1727C4.36689 13.1727 1.82708 10.6329 1.82708 7.49988ZM10.1589 5.53774C10.3178 5.31191 10.2636 5.00001 10.0378 4.84109C9.81194 4.68217 9.50004 4.73642 9.34112 4.96225L6.51977 8.97154L5.35681 7.78706C5.16334 7.59002 4.84677 7.58711 4.64973 7.78058C4.45268 7.97404 4.44978 8.29061 4.64325 8.48765L6.22658 10.1003C6.33054 10.2062 6.47617 10.2604 6.62407 10.2483C6.77197 10.2363 6.90686 10.1591 6.99226 10.0377L10.1589 5.53774Z"
																fill="currentColor"
																fill-rule="evenodd"
																clip-rule="evenodd"
															></path></svg
														>Search exact work
													</Button>
												{:else}
													<Button
														size="sm"
														variant="ghost"
														class="text text-muted-foreground"
														on:click={autocompleteWork}
													>
														<!-- <svg
															class="margin-right mr-2 dark:text-lime-400"
															width="15"
															height="15"
															viewBox="0 0 15 15"
															fill="none"
															xmlns="http://www.w3.org/2000/svg"
															><path
																d="M3 2.5C3 2.22386 3.22386 2 3.5 2H9.08579C9.21839 2 9.34557 2.05268 9.43934 2.14645L11.8536 4.56066C11.9473 4.65443 12 4.78161 12 4.91421V12.5C12 12.7761 11.7761 13 11.5 13H3.5C3.22386 13 3 12.7761 3 12.5V2.5ZM3.5 1C2.67157 1 2 1.67157 2 2.5V12.5C2 13.3284 2.67157 14 3.5 14H11.5C12.3284 14 13 13.3284 13 12.5V4.91421C13 4.51639 12.842 4.13486 12.5607 3.85355L10.1464 1.43934C9.86514 1.15804 9.48361 1 9.08579 1H3.5ZM4.5 4C4.22386 4 4 4.22386 4 4.5C4 4.77614 4.22386 5 4.5 5H7.5C7.77614 5 8 4.77614 8 4.5C8 4.22386 7.77614 4 7.5 4H4.5ZM4.5 7C4.22386 7 4 7.22386 4 7.5C4 7.77614 4.22386 8 4.5 8H10.5C10.7761 8 11 7.77614 11 7.5C11 7.22386 10.7761 7 10.5 7H4.5ZM4.5 10C4.22386 10 4 10.2239 4 10.5C4 10.7761 4.22386 11 4.5 11H10.5C10.7761 11 11 10.7761 11 10.5C11 10.2239 10.7761 10 10.5 10H4.5Z"
																fill="currentColor"
																fill-rule="evenodd"
																clip-rule="evenodd"
															></path></svg -->
														<svg
															class="margin-right mr-2"
															width="15"
															height="15"
															viewBox="0 0 15 15"
															fill="none"
															xmlns="http://www.w3.org/2000/svg"
															><path
																d="M0.877075 7.49991C0.877075 3.84222 3.84222 0.877075 7.49991 0.877075C11.1576 0.877075 14.1227 3.84222 14.1227 7.49991C14.1227 11.1576 11.1576 14.1227 7.49991 14.1227C3.84222 14.1227 0.877075 11.1576 0.877075 7.49991ZM7.49991 1.82708C4.36689 1.82708 1.82708 4.36689 1.82708 7.49991C1.82708 10.6329 4.36689 13.1727 7.49991 13.1727C10.6329 13.1727 13.1727 10.6329 13.1727 7.49991C13.1727 4.36689 10.6329 1.82708 7.49991 1.82708Z"
																fill="currentColor"
																fill-rule="evenodd"
																clip-rule="evenodd"
															></path></svg
														>Search exact work
													</Button>
												{/if}
											</Dialog.Trigger>
											<Dialog.Content class="padding p-0">
												<Command.Root>
													<div class="flex items-center border-b px-3">
														<svg
															class="mr-2 h-4 w-4 shrink-0 opacity-50"
															viewBox="0 0 15 15"
															fill="none"
															xmlns="http://www.w3.org/2000/svg"
															><path
																d="M10 6.5C10 8.433 8.433 10 6.5 10C4.567 10 3 8.433 3 6.5C3 4.567 4.567 3 6.5 3C8.433 3 10 4.567 10 6.5ZM9.30884 10.0159C8.53901 10.6318 7.56251 11 6.5 11C4.01472 11 2 8.98528 2 6.5C2 4.01472 4.01472 2 6.5 2C8.98528 2 11 4.01472 11 6.5C11 7.56251 10.6318 8.53901 10.0159 9.30884L12.8536 12.1464C13.0488 12.3417 13.0488 12.6583 12.8536 12.8536C12.6583 13.0488 12.3417 13.0488 12.1464 12.8536L9.30884 10.0159Z"
																fill="currentColor"
																fill-rule="evenodd"
																clip-rule="evenodd"
															></path></svg
														>
														<input
															class="flex h-11 w-full rounded-md bg-transparent py-3 text-sm outline-none placeholder:text-muted-foreground disabled:cursor-not-allowed disabled:opacity-50"
															type="text"
															bind:value={workTitle}
															on:input={autocompleteWork}
															placeholder="Type work title"
														/>
													</div>
													{#if workId !== ''}
														<Command.Group heading="Actions">
															<Command.Item>
																<button
																	class="padding p-.5 relative flex w-full cursor-default select-none items-center rounded-sm text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																	on:click={clearWork}
																>
																	<p style="display: inline-flex; align-items:center">
																		<svg
																			class="margin-right mr-2"
																			width="15"
																			height="15"
																			viewBox="0 0 15 15"
																			fill="none"
																			xmlns="http://www.w3.org/2000/svg"
																			><path
																				d="M4.85355 2.14645C5.04882 2.34171 5.04882 2.65829 4.85355 2.85355L3.70711 4H9C11.4853 4 13.5 6.01472 13.5 8.5C13.5 10.9853 11.4853 13 9 13H5C4.72386 13 4.5 12.7761 4.5 12.5C4.5 12.2239 4.72386 12 5 12H9C10.933 12 12.5 10.433 12.5 8.5C12.5 6.567 10.933 5 9 5H3.70711L4.85355 6.14645C5.04882 6.34171 5.04882 6.65829 4.85355 6.85355C4.65829 7.04882 4.34171 7.04882 4.14645 6.85355L2.14645 4.85355C1.95118 4.65829 1.95118 4.34171 2.14645 4.14645L4.14645 2.14645C4.34171 1.95118 4.65829 1.95118 4.85355 2.14645Z"
																				fill="currentColor"
																				fill-rule="evenodd"
																				clip-rule="evenodd"
																			></path></svg
																		>
																		Reset
																	</p>
																</button>
															</Command.Item>
														</Command.Group>
													{:else}
														<div></div>
													{/if}
													<Command.List>
														<Command.Group heading="Works">
															<Command.Empty>No results found.</Command.Empty>
															{#each autocomplete_works as work}
																{#if workId == work.id.substring('https://openalex.org/'.length)}
																	<Command.Item>
																		<button
																			class="text padding p-.5 text justify cursor-default select-none justify-items-start rounded-sm text-left text-sm text-muted-foreground outline-none aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																			on:click={clearWork}
																		>
																			{work.display_name} <br />
																			<span class="text text-xs">
																				{work.hint}
																			</span>
																		</button>
																	</Command.Item>
																{:else}
																	<Command.Item>
																		<button
																			class="padding p-.5 text justify cursor-default select-none justify-items-start rounded-sm text-left text-sm outline-none hover:text-blue-500 aria-selected:bg-accent aria-selected:text-accent-foreground data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
																			on:click={() =>
																				setWork(
																					work.id.substring('https://openalex.org/'.length),
																					work.display_name
																				)}
																		>
																			{work.display_name} <br />
																			<span class="text text-xs">
																				{work.hint}
																			</span>
																		</button>
																	</Command.Item>
																{/if}
															{/each}
														</Command.Group>
														<Command.Separator />
													</Command.List>
													{#if workId !== ''}
														<div
															class="
														padding relative flex cursor-default select-none items-center
														rounded-sm p-2 text-sm outline-none hover:text-blue-500
														aria-selected:bg-accent aria-selected:text-accent-foreground
														data-[disabled]:pointer-events-none data-[disabled]:opacity-50"
														>
															<p
																style="display: inline-flex; align-items:center"
																class="text-sm dark:text-lime-400"
															>
																<svg
																	class="margin-right mr-2"
																	width="15"
																	height="15"
																	viewBox="0 0 15 15"
																	fill="none"
																	xmlns="http://www.w3.org/2000/svg"
																	><path
																		d="M7.49991 0.877045C3.84222 0.877045 0.877075 3.84219 0.877075 7.49988C0.877075 11.1575 3.84222 14.1227 7.49991 14.1227C11.1576 14.1227 14.1227 11.1575 14.1227 7.49988C14.1227 3.84219 11.1576 0.877045 7.49991 0.877045ZM1.82708 7.49988C1.82708 4.36686 4.36689 1.82704 7.49991 1.82704C10.6329 1.82704 13.1727 4.36686 13.1727 7.49988C13.1727 10.6329 10.6329 13.1727 7.49991 13.1727C4.36689 13.1727 1.82708 10.6329 1.82708 7.49988ZM10.1589 5.53774C10.3178 5.31191 10.2636 5.00001 10.0378 4.84109C9.81194 4.68217 9.50004 4.73642 9.34112 4.96225L6.51977 8.97154L5.35681 7.78706C5.16334 7.59002 4.84677 7.58711 4.64973 7.78058C4.45268 7.97404 4.44978 8.29061 4.64325 8.48765L6.22658 10.1003C6.33054 10.2062 6.47617 10.2604 6.62407 10.2483C6.77197 10.2363 6.90686 10.1591 6.99226 10.0377L10.1589 5.53774Z"
																		fill="currentColor"
																		fill-rule="evenodd"
																		clip-rule="evenodd"
																	></path></svg
																>
																{#if total <= 0}
																	Filter active: no results found
																{:else}
																	{#each items as work}
																		Filter active: {work.id.substring('https://'.length)}
																	{/each}
																{/if}
															</p>
														</div>
													{:else}
														<div></div>
													{/if}
												</Command.Root>
											</Dialog.Content>
										</div>
									</Dialog.Root>

									{#if fulltext !== ''}
										<Button size="sm" variant="ghost" on:click={clear_fulltext}>
											<svg
												class="margin-right mr-2 dark:text-lime-400"
												width="15"
												height="15"
												viewBox="0 0 15 15"
												fill="none"
												xmlns="http://www.w3.org/2000/svg"
												><path
													d="M7.49991 0.877045C3.84222 0.877045 0.877075 3.84219 0.877075 7.49988C0.877075 11.1575 3.84222 14.1227 7.49991 14.1227C11.1576 14.1227 14.1227 11.1575 14.1227 7.49988C14.1227 3.84219 11.1576 0.877045 7.49991 0.877045ZM1.82708 7.49988C1.82708 4.36686 4.36689 1.82704 7.49991 1.82704C10.6329 1.82704 13.1727 4.36686 13.1727 7.49988C13.1727 10.6329 10.6329 13.1727 7.49991 13.1727C4.36689 13.1727 1.82708 10.6329 1.82708 7.49988ZM10.1589 5.53774C10.3178 5.31191 10.2636 5.00001 10.0378 4.84109C9.81194 4.68217 9.50004 4.73642 9.34112 4.96225L6.51977 8.97154L5.35681 7.78706C5.16334 7.59002 4.84677 7.58711 4.64973 7.78058C4.45268 7.97404 4.44978 8.29061 4.64325 8.48765L6.22658 10.1003C6.33054 10.2062 6.47617 10.2604 6.62407 10.2483C6.77197 10.2363 6.90686 10.1591 6.99226 10.0377L10.1589 5.53774Z"
													fill="currentColor"
													fill-rule="evenodd"
													clip-rule="evenodd"
												></path></svg
											>Has fulltext
										</Button>
									{:else}
										<Button
											size="sm"
											variant="ghost"
											class="text text-muted-foreground"
											on:click={has_fulltext}
										>
											<svg
												class="margin-right mr-2"
												width="15"
												height="15"
												viewBox="0 0 15 15"
												fill="none"
												xmlns="http://www.w3.org/2000/svg"
												><path
													d="M0.877075 7.49991C0.877075 3.84222 3.84222 0.877075 7.49991 0.877075C11.1576 0.877075 14.1227 3.84222 14.1227 7.49991C14.1227 11.1576 11.1576 14.1227 7.49991 14.1227C3.84222 14.1227 0.877075 11.1576 0.877075 7.49991ZM7.49991 1.82708C4.36689 1.82708 1.82708 4.36689 1.82708 7.49991C1.82708 10.6329 4.36689 13.1727 7.49991 13.1727C10.6329 13.1727 13.1727 10.6329 13.1727 7.49991C13.1727 4.36689 10.6329 1.82708 7.49991 1.82708Z"
													fill="currentColor"
													fill-rule="evenodd"
													clip-rule="evenodd"
												></path></svg
											>Has fulltext
										</Button>
									{/if}

									<t class="text-sm font-medium text-muted-foreground">
										<span class="font-bold">
											{total}
										</span>
										{#if total > 1 || total == 0}
											results
										{:else}
											result
										{/if}
										<!-- {#if authorId !== ''}
											by
											<Button variant="ghost" size="sm" class="text text-md" on:click={clearAuthor}>
												{authorName}
												<svg
													class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
													viewBox="0 0 15 15"
													fill="none"
													xmlns="http://www.w3.org/2000/svg"
													><path
														d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
														fill="currentColor"
														fill-rule="evenodd"
														clip-rule="evenodd"
													></path></svg
												>
											</Button>
										{:else}
											<div></div>
										{/if} -->
									</t>
									<!-- <t class="text-sm font-medium text-muted-foreground">
										{#if sourceId !== ''}
											from
											<Button variant="ghost" size="sm" class="text text-md" on:click={clearSource}>
												{sourceName}
												<svg
													class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
													viewBox="0 0 15 15"
													fill="none"
													xmlns="http://www.w3.org/2000/svg"
													><path
														d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
														fill="currentColor"
														fill-rule="evenodd"
														clip-rule="evenodd"
													></path></svg
												>
											</Button>
										{:else}
											<div></div>
										{/if}
									</t> -->
									<!-- <t class="text-sm font-medium text-muted-foreground">
										{#if pubYear !== ''}
											published in
											<Button
												variant="ghost"
												size="sm"
												class="text text-md"
												on:click={clearPublished}
											>
												{pubYear}
												<svg
													class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
													viewBox="0 0 15 15"
													fill="none"
													xmlns="http://www.w3.org/2000/svg"
													><path
														d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
														fill="currentColor"
														fill-rule="evenodd"
														clip-rule="evenodd"
													></path></svg
												>
											</Button>
										{:else}
											<div></div>
										{/if}
									</t> -->
								</div>
								<div
									class="ml-auto inline-flex h-10 items-center justify-between space-x-2 whitespace-nowrap rounded-md bg-background px-4 py-2 text-sm font-medium ring-offset-background"
								>
									{#if cursor == null || cursor == '*'}
										<Button
											style="visibility: hidden;"
											variant="ghost"
											size="icon"
											on:click={prevPage}
										>
											prev
										</Button>
									{:else}
										<Button variant="ghost" size="icon" on:click={prevPage}>
											<svg
												xmlns="http://www.w3.org/2000/svg"
												width="24"
												height="24"
												viewBox="0 0 24 24"
												fill="none"
												stroke="currentColor"
												stroke-width="2"
												stroke-linecap="round"
												stroke-linejoin="round"
												class="lucide lucide-chevron-left"><path d="m15 18-6-6 6-6" /></svg
											>
										</Button>
									{/if}
									{#if items.length < pageSize}
										<Button
											style="visibility: hidden"
											variant="ghost"
											size="icon"
											on:click={nextPage}
										>
											next
										</Button>
									{:else}
										<Button variant="ghost" size="icon" on:click={nextPage}>
											<svg
												xmlns="http://www.w3.org/2000/svg"
												width="24"
												height="24"
												viewBox="0 0 24 24"
												fill="none"
												stroke="currentColor"
												stroke-width="2"
												stroke-linecap="round"
												stroke-linejoin="round"
												class="lucide lucide-chevron-right"><path d="m9 18 6-6-6-6" /></svg
											>
										</Button>
									{/if}
								</div>
							</div>
							<Table.Root>
								<Table.Caption>
									<div style="display: flex; justify-content: space-between; align-items: baseline">
										{#if cursor == null || cursor == '*'}
											<div style="visibility: hidden;">
												<Button variant="ghost">
													<svg
														xmlns="http://www.w3.org/2000/svg"
														width="24"
														height="24"
														viewBox="0 0 24 24"
														fill="none"
														stroke="currentColor"
														stroke-width="2"
														stroke-linecap="round"
														stroke-linejoin="round"
														class="lucide lucide-chevron-left"><path d="m15 18-6-6 6-6" /></svg
													>
												</Button>
											</div>
										{:else}
											<div>
												<Button
													variant="ghost"
													on:click={prevPage}
													disabled={cursor == null || cursor == '*'}
												>
													<svg
														xmlns="http://www.w3.org/2000/svg"
														width="24"
														height="24"
														viewBox="0 0 24 24"
														fill="none"
														stroke="currentColor"
														stroke-width="2"
														stroke-linecap="round"
														stroke-linejoin="round"
														class="lucide lucide-chevron-left"><path d="m15 18-6-6 6-6" /></svg
													>
												</Button>
											</div>
										{/if}
										{#if items.length < pageSize}
											<div style="visibility: hidden;">
												<Button variant="ghost">
													<svg
														xmlns="http://www.w3.org/2000/svg"
														width="24"
														height="24"
														viewBox="0 0 24 24"
														fill="none"
														stroke="currentColor"
														stroke-width="2"
														stroke-linecap="round"
														stroke-linejoin="round"
														class="lucide lucide-chevron-right"><path d="m9 18 6-6-6-6" /></svg
													>
												</Button>
											</div>
										{:else}
											<div>
												<Button variant="ghost" on:click={nextPage} disabled={!next}>
													<svg
														xmlns="http://www.w3.org/2000/svg"
														width="24"
														height="24"
														viewBox="0 0 24 24"
														fill="none"
														stroke="currentColor"
														stroke-width="2"
														stroke-linecap="round"
														stroke-linejoin="round"
														class="lucide lucide-chevron-right"><path d="m9 18 6-6-6-6" /></svg
													>
												</Button>
											</div>
										{/if}
									</div>
								</Table.Caption>
								<Table.Footer>
									<Table.Row></Table.Row>
								</Table.Footer>
								<Table.Header>
									<Table.Row>
										<Table.Head>
											{#if workType == ''}
												Works
											{:else}
												<Button variant="ghost" size="sm" class="text text-md" on:click={clearType}>
													{workType}
													<svg
														class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
														viewBox="0 0 15 15"
														fill="none"
														xmlns="http://www.w3.org/2000/svg"
														><path
															d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
															fill="currentColor"
															fill-rule="evenodd"
															clip-rule="evenodd"
														></path></svg
													>
												</Button>
											{/if}
											<!-- Works -->
										</Table.Head>
										<Table.Head class="text-right">
											{#if statusName == ''}
												Status
											{:else}
												<Button
													variant="ghost"
													size="sm"
													class="text text-md"
													on:click={clearStatus}
												>
													{statusName}
													<svg
														class="margin-left ml-2 h-[1.2rem] w-[1.2rem]"
														viewBox="0 0 15 15"
														fill="none"
														xmlns="http://www.w3.org/2000/svg"
														><path
															d="M11.7816 4.03157C12.0062 3.80702 12.0062 3.44295 11.7816 3.2184C11.5571 2.99385 11.193 2.99385 10.9685 3.2184L7.50005 6.68682L4.03164 3.2184C3.80708 2.99385 3.44301 2.99385 3.21846 3.2184C2.99391 3.44295 2.99391 3.80702 3.21846 4.03157L6.68688 7.49999L3.21846 10.9684C2.99391 11.193 2.99391 11.557 3.21846 11.7816C3.44301 12.0061 3.80708 12.0061 4.03164 11.7816L7.50005 8.31316L10.9685 11.7816C11.193 12.0061 11.5571 12.0061 11.7816 11.7816C12.0062 11.557 12.0062 11.193 11.7816 10.9684L8.31322 7.49999L11.7816 4.03157Z"
															fill="currentColor"
															fill-rule="evenodd"
															clip-rule="evenodd"
														></path></svg
													>
												</Button>
											{/if}
											<!-- Status -->
										</Table.Head>
									</Table.Row>
								</Table.Header>
								<Table.Body>
									{#each items as work}
										<Table.Row>
											<Table.Cell class="font-medium">
												<a href={work.doi} target="_blank" rel="noreferrer noopener">
													{work.title}
												</a>
												<div style="display: flex; " class="text-muted-foreground">
													<div>
														<!-- class="text-md" -->
														<Button
															variant="ghost"
															size="sm"
															class="text text-md"
															on:click={() => setPubYear(work.publication_year)}
															>{work.publication_date + ''}</Button
														>
														{#each work.authorships as author}
															<!-- class="text-md" -->
															<Button
																variant="ghost"
																size="sm"
																class="text text-md"
																on:click={() =>
																	setAuthor(
																		author.author.id.substring('https://openalex.org/'.length),
																		author.author.display_name
																	)}>{author.author.display_name}</Button
															>
														{/each}
														{#if work.primary_location !== null}
															{#if work.primary_location.source !== null}
																<!-- class="text-md" -->
																<Button
																	variant="ghost"
																	size="sm"
																	class="text text-md"
																	style="font-style: italic;"
																	on:click={() =>
																		setSource(
																			work.primary_location.source.id.substring(
																				'https://openalex.org/'.length
																			),
																			work.primary_location.source.display_name
																		)}>{work.primary_location.source.display_name}</Button
																>
															{:else}
																<div></div>
															{/if}
														{:else}
															<div></div>
														{/if}
													</div>
												</div>
												<div style="display: flex; ">
													{#if work.cited_by_count > 0}
														<div>
															<!-- class="text-md font-semibold" -->
															<Button
																class="font-semibold"
																variant="ghost"
																size="sm"
																on:click={() =>
																	setCites(work.id.substring('https://openalex.org/'.length))}
															>
																Cited by {work.cited_by_count}
															</Button>
															<!-- <Button
																class="font-semibold"
																variant="ghost"
																size="sm"
																href="https://openalex.org/works?page=1&filter=cites:{work.id.substring(
																	'https://openalex.org/'.length
																)}"
																target="_blank"
																rel="noreferrer noopener"
															>
																Cited by {work.cited_by_count}
															</Button> -->
														</div>
													{:else}
														<div></div>
													{/if}
													{#if work.primary_location !== null}
														{#if work.primary_location.pdf_url !== null}
															<div>
																<!-- class="text-md font-semibold" -->
																<Button
																	class="font-semibold"
																	variant="ghost"
																	size="sm"
																	href={work.primary_location.pdf_url}
																	target="_blank"
																	rel="noreferrer noopener"
																>
																	<svg
																		class="margin-right mr-2"
																		width="15"
																		height="15"
																		viewBox="0 0 15 15"
																		fill="none"
																		xmlns="http://www.w3.org/2000/svg"
																		><path
																			d="M3 2.5C3 2.22386 3.22386 2 3.5 2H9.08579C9.21839 2 9.34557 2.05268 9.43934 2.14645L11.8536 4.56066C11.9473 4.65443 12 4.78161 12 4.91421V12.5C12 12.7761 11.7761 13 11.5 13H3.5C3.22386 13 3 12.7761 3 12.5V2.5ZM3.5 1C2.67157 1 2 1.67157 2 2.5V12.5C2 13.3284 2.67157 14 3.5 14H11.5C12.3284 14 13 13.3284 13 12.5V4.91421C13 4.51639 12.842 4.13486 12.5607 3.85355L10.1464 1.43934C9.86514 1.15804 9.48361 1 9.08579 1H3.5ZM4.5 4C4.22386 4 4 4.22386 4 4.5C4 4.77614 4.22386 5 4.5 5H7.5C7.77614 5 8 4.77614 8 4.5C8 4.22386 7.77614 4 7.5 4H4.5ZM4.5 7C4.22386 7 4 7.22386 4 7.5C4 7.77614 4.22386 8 4.5 8H10.5C10.7761 8 11 7.77614 11 7.5C11 7.22386 10.7761 7 10.5 7H4.5ZM4.5 10C4.22386 10 4 10.2239 4 10.5C4 10.7761 4.22386 11 4.5 11H10.5C10.7761 11 11 10.7761 11 10.5C11 10.2239 10.7761 10 10.5 10H4.5Z"
																			fill="currentColor"
																			fill-rule="evenodd"
																			clip-rule="evenodd"
																		></path></svg
																	>
																	PDF
																</Button>
															</div>
														{:else}
															<div></div>
														{/if}
													{:else}
														<div></div>
													{/if}
												</div>
											</Table.Cell>
											<Table.Cell class="text-right">
												<div>
													<Sheet.Root>
														<Sheet.Trigger asChild let:builder>
															<Button
																builders={[builder]}
																variant="ghost"
																style="text-transform: uppercase"
															>
																<!-- class="text-md" -->
																{#if work.open_access.oa_status == 'closed'}
																	<t class="text-muted-foreground">
																		{work.open_access.oa_status}
																	</t>
																{:else if work.open_access.oa_status == 'diamond'}
																	<svg
																		xmlns="http://www.w3.org/2000/svg"
																		viewBox="0 0 24 24"
																		fill="none"
																		stroke="currentColor"
																		stroke-width="2"
																		stroke-linecap="round"
																		stroke-linejoin="round"
																		class="lucide lucide-diamond mr-2 h-[1rem] w-[1rem] dark:stroke-lime-400"
																		><path
																			d="M2.7 10.3a2.41 2.41 0 0 0 0 3.41l7.59 7.59a2.41 2.41 0 0 0 3.41 0l7.59-7.59a2.41 2.41 0 0 0 0-3.41l-7.59-7.59a2.41 2.41 0 0 0-3.41 0Z"
																		/></svg
																	>
																	{work.open_access.oa_status}
																{:else}
																	{work.open_access.oa_status}
																{/if}
																<svg
																	xmlns="http://www.w3.org/2000/svg"
																	width="24"
																	height="24"
																	viewBox="0 0 24 24"
																	fill="none"
																	stroke="currentColor"
																	stroke-width="2"
																	stroke-linecap="round"
																	stroke-linejoin="round"
																	class="lucide lucide-chevron-right margin-left ml-2"
																	><path d="m9 18 6-6-6-6" /></svg
																>
															</Button>
														</Sheet.Trigger>
														<Sheet.Content side="right" class="h-screen max-h-max overflow-y-auto">
															<Sheet.Header>
																<Sheet.Title>{work.title}</Sheet.Title>
																<Sheet.Description>
																	<Button
																		variant="ghost"
																		size="sm"
																		class="text text-md"
																		href="https://openalex.org/works/{work.id.substring(
																			'https://openalex.org/'.length
																		)}"
																		target="_blank"
																		rel="noreferrer noopenner"
																	>
																		{work.type}
																	</Button>
																</Sheet.Description>
																<div style="display:inline-flex; " class="justify space-x-1">
																	<div>
																		{#if work.open_access.oa_status == 'closed'}
																			<Button
																				variant="ghost"
																				size="sm"
																				class="text text-md"
																				href={work.doi}
																				target="_blank"
																				rel="noreferrer noopener"
																			>
																				HTML
																				<svg
																					class="margin-left ml-2"
																					width="15"
																					height="15"
																					viewBox="0 0 15 15"
																					fill="none"
																					xmlns="http://www.w3.org/2000/svg"
																					><path
																						d="M5 4.63601C5 3.76031 5.24219 3.1054 5.64323 2.67357C6.03934 2.24705 6.64582 1.9783 7.5014 1.9783C8.35745 1.9783 8.96306 2.24652 9.35823 2.67208C9.75838 3.10299 10 3.75708 10 4.63325V5.99999H5V4.63601ZM4 5.99999V4.63601C4 3.58148 4.29339 2.65754 4.91049 1.99307C5.53252 1.32329 6.42675 0.978302 7.5014 0.978302C8.57583 0.978302 9.46952 1.32233 10.091 1.99162C10.7076 2.65557 11 3.57896 11 4.63325V5.99999H12C12.5523 5.99999 13 6.44771 13 6.99999V13C13 13.5523 12.5523 14 12 14H3C2.44772 14 2 13.5523 2 13V6.99999C2 6.44771 2.44772 5.99999 3 5.99999H4ZM3 6.99999H12V13H3V6.99999Z"
																						fill="currentColor"
																						fill-rule="evenodd"
																						clip-rule="evenodd"
																					></path></svg
																				>
																			</Button>
																		{:else}
																			<Button
																				variant="default"
																				size="sm"
																				class="text text-md"
																				href={work.doi}
																				target="_blank"
																				rel="noreferrer noopener"
																			>
																				HTML
																				<svg
																					class="margin-left ml-2"
																					width="15"
																					height="15"
																					viewBox="0 0 15 15"
																					fill="none"
																					xmlns="http://www.w3.org/2000/svg"
																					><path
																						d="M3 2C2.44772 2 2 2.44772 2 3V12C2 12.5523 2.44772 13 3 13H12C12.5523 13 13 12.5523 13 12V8.5C13 8.22386 12.7761 8 12.5 8C12.2239 8 12 8.22386 12 8.5V12H3V3L6.5 3C6.77614 3 7 2.77614 7 2.5C7 2.22386 6.77614 2 6.5 2H3ZM12.8536 2.14645C12.9015 2.19439 12.9377 2.24964 12.9621 2.30861C12.9861 2.36669 12.9996 2.4303 13 2.497L13 2.5V2.50049V5.5C13 5.77614 12.7761 6 12.5 6C12.2239 6 12 5.77614 12 5.5V3.70711L6.85355 8.85355C6.65829 9.04882 6.34171 9.04882 6.14645 8.85355C5.95118 8.65829 5.95118 8.34171 6.14645 8.14645L11.2929 3H9.5C9.22386 3 9 2.77614 9 2.5C9 2.22386 9.22386 2 9.5 2H12.4999H12.5C12.5678 2 12.6324 2.01349 12.6914 2.03794C12.7504 2.06234 12.8056 2.09851 12.8536 2.14645Z"
																						fill="currentColor"
																						fill-rule="evenodd"
																						clip-rule="evenodd"
																					></path></svg
																				>
																			</Button>
																		{/if}
																	</div>
																	{#if work.primary_location !== null}
																		{#if work.primary_location.pdf_url !== null}
																			<div>
																				<Button
																					variant="secondary"
																					size="sm"
																					class="text text-md"
																					href={work.primary_location.pdf_url}
																					target="_blank"
																					rel="noreferrer noopener"
																				>
																					<svg
																						class="margin-right mr-2"
																						width="15"
																						height="15"
																						viewBox="0 0 15 15"
																						fill="none"
																						xmlns="http://www.w3.org/2000/svg"
																						><path
																							d="M3 2.5C3 2.22386 3.22386 2 3.5 2H9.08579C9.21839 2 9.34557 2.05268 9.43934 2.14645L11.8536 4.56066C11.9473 4.65443 12 4.78161 12 4.91421V12.5C12 12.7761 11.7761 13 11.5 13H3.5C3.22386 13 3 12.7761 3 12.5V2.5ZM3.5 1C2.67157 1 2 1.67157 2 2.5V12.5C2 13.3284 2.67157 14 3.5 14H11.5C12.3284 14 13 13.3284 13 12.5V4.91421C13 4.51639 12.842 4.13486 12.5607 3.85355L10.1464 1.43934C9.86514 1.15804 9.48361 1 9.08579 1H3.5ZM4.5 4C4.22386 4 4 4.22386 4 4.5C4 4.77614 4.22386 5 4.5 5H7.5C7.77614 5 8 4.77614 8 4.5C8 4.22386 7.77614 4 7.5 4H4.5ZM4.5 7C4.22386 7 4 7.22386 4 7.5C4 7.77614 4.22386 8 4.5 8H10.5C10.7761 8 11 7.77614 11 7.5C11 7.22386 10.7761 7 10.5 7H4.5ZM4.5 10C4.22386 10 4 10.2239 4 10.5C4 10.7761 4.22386 11 4.5 11H10.5C10.7761 11 11 10.7761 11 10.5C11 10.2239 10.7761 10 10.5 10H4.5Z"
																							fill="currentColor"
																							fill-rule="evenodd"
																							clip-rule="evenodd"
																						></path></svg
																					>
																					PDF
																				</Button>
																			</div>
																		{:else}
																			<div></div>
																		{/if}
																	{:else}
																		<div></div>
																	{/if}
																	{#if work.doi !== null}
																		<div>
																			<Button
																				variant="outline"
																				size="sm"
																				class="text text-md"
																				on:click={() => getCitations(work.doi)}
																			>
																				<!-- href="https://api.citeas.org/product/{work.doi}"
																			target="_blank"
																			rel="noreferrer noopenner" -->
																				Cite
																				<svg
																					class="margin ml-2"
																					width="15"
																					height="15"
																					viewBox="0 0 15 15"
																					fill="none"
																					xmlns="http://www.w3.org/2000/svg"
																					><path
																						d="M5 2V1H10V2H5ZM4.75 0C4.33579 0 4 0.335786 4 0.75V1H3.5C2.67157 1 2 1.67157 2 2.5V12.5C2 13.3284 2.67157 14 3.5 14H7V13H3.5C3.22386 13 3 12.7761 3 12.5V2.5C3 2.22386 3.22386 2 3.5 2H4V2.25C4 2.66421 4.33579 3 4.75 3H10.25C10.6642 3 11 2.66421 11 2.25V2H11.5C11.7761 2 12 2.22386 12 2.5V7H13V2.5C13 1.67157 12.3284 1 11.5 1H11V0.75C11 0.335786 10.6642 0 10.25 0H4.75ZM9 8.5C9 8.77614 8.77614 9 8.5 9C8.22386 9 8 8.77614 8 8.5C8 8.22386 8.22386 8 8.5 8C8.77614 8 9 8.22386 9 8.5ZM10.5 9C10.7761 9 11 8.77614 11 8.5C11 8.22386 10.7761 8 10.5 8C10.2239 8 10 8.22386 10 8.5C10 8.77614 10.2239 9 10.5 9ZM13 8.5C13 8.77614 12.7761 9 12.5 9C12.2239 9 12 8.77614 12 8.5C12 8.22386 12.2239 8 12.5 8C12.7761 8 13 8.22386 13 8.5ZM14.5 9C14.7761 9 15 8.77614 15 8.5C15 8.22386 14.7761 8 14.5 8C14.2239 8 14 8.22386 14 8.5C14 8.77614 14.2239 9 14.5 9ZM15 10.5C15 10.7761 14.7761 11 14.5 11C14.2239 11 14 10.7761 14 10.5C14 10.2239 14.2239 10 14.5 10C14.7761 10 15 10.2239 15 10.5ZM14.5 13C14.7761 13 15 12.7761 15 12.5C15 12.2239 14.7761 12 14.5 12C14.2239 12 14 12.2239 14 12.5C14 12.7761 14.2239 13 14.5 13ZM14.5 15C14.7761 15 15 14.7761 15 14.5C15 14.2239 14.7761 14 14.5 14C14.2239 14 14 14.2239 14 14.5C14 14.7761 14.2239 15 14.5 15ZM8.5 11C8.77614 11 9 10.7761 9 10.5C9 10.2239 8.77614 10 8.5 10C8.22386 10 8 10.2239 8 10.5C8 10.7761 8.22386 11 8.5 11ZM9 12.5C9 12.7761 8.77614 13 8.5 13C8.22386 13 8 12.7761 8 12.5C8 12.2239 8.22386 12 8.5 12C8.77614 12 9 12.2239 9 12.5ZM8.5 15C8.77614 15 9 14.7761 9 14.5C9 14.2239 8.77614 14 8.5 14C8.22386 14 8 14.2239 8 14.5C8 14.7761 8.22386 15 8.5 15ZM11 14.5C11 14.7761 10.7761 15 10.5 15C10.2239 15 10 14.7761 10 14.5C10 14.2239 10.2239 14 10.5 14C10.7761 14 11 14.2239 11 14.5ZM12.5 15C12.7761 15 13 14.7761 13 14.5C13 14.2239 12.7761 14 12.5 14C12.2239 14 12 14.2239 12 14.5C12 14.7761 12.2239 15 12.5 15Z"
																						fill="currentColor"
																						fill-rule="evenodd"
																						clip-rule="evenodd"
																					></path></svg
																				>
																			</Button>
																		</div>
																	{:else}
																		<div></div>
																	{/if}
																	<div>
																		<Button
																			variant="outline"
																			size="sm"
																			class="text text-md font-mono font-semibold"
																			href="https://api.openalex.org/{work.id.substring(
																				'https://openalex.org/'.length
																			)}"
																			target="_blank"
																			rel="noreferrer noopenner"
																		>
																			API
																		</Button>
																	</div>
																</div>
															</Sheet.Header>
															<div class="grid gap-4 py-4">
																<div>
																	<!-- <hr style="margin-bottom: .5rem" /> -->
																	{#if work.abstract_inverted_index !== null}
																		<Accordion.Root class="margin mb-4">
																			<Accordion.Item value="item-1">
																				<Accordion.Trigger>Abstract</Accordion.Trigger>
																				<Accordion.Content>
																					<!-- <span class="font font-bold"> Abstract: </span> -->
																					{generateParagraph(work.abstract_inverted_index)}
																				</Accordion.Content>
																			</Accordion.Item>
																		</Accordion.Root>
																		<!-- <br /> -->
																	{:else}
																		<t class="text text-muted-foreground"></t>
																	{/if}
																	<span style="font-weight: bold;"> Published: </span>
																	{work.publication_date}
																	<br />
																	{#if work.primary_location !== null}
																		{#if work.primary_location.source !== null}
																			<span style="font-weight: bold;"> Source: </span>
																			<a
																				href={work.primary_location.source.id}
																				target="_blank"
																				rel="noreferrer noopener"
																				class="text-blue-500 hover:underline"
																			>
																				{work.primary_location.source.display_name}
																			</a> <br />
																		{:else}
																			<div></div>
																		{/if}
																	{:else}
																		<div></div>
																	{/if}
																	<span style="font-weight: bold;"> Authors: </span>
																	{#each work.authorships as author}
																		<a
																			href={author.author.id}
																			target="_blank"
																			rel="noreferrer noopener"
																			class="space-around -flex items-center gap-1 rounded-md border border-transparent px-2.5 py-0.5 text-blue-500 transition-colors hover:underline focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2"
																		>
																			{author.author.display_name + ' '}
																		</a>
																	{/each} <br />
																	<span style="font-weight: bold;"> Institutions: </span>
																	{#each work.authorships as author}
																		{#if author.institutions[0] !== undefined}
																			<!-- <a
																				href={author.institutions[0].id}
																				target="_blank"
																				rel="noreferrer noopener"
																				class="space-around -flex items-center gap-1 rounded-md border border-transparent px-2.5 py-0.5 text-blue-500 transition-colors hover:underline focus:outline-none focus:ring-2 focus:ring-ring focus:ring-offset-2"
																			>
																				{author.institutions[0].display_name}
																			</a> -->
																			<!-- <br /> -->
																			<Sheet.Close asChild let:builder>
																				<Button
																					builders={[builder]}
																					variant="ghost"
																					size="sm"
																					class="text text-base font-normal text-blue-500"
																					on:click={() =>
																						setInstitute(
																							author.institutions[0].id.substring(
																								'https://openalex.org/'.length
																							),
																							author.institutions[0].display_name
																						)}
																					>{author.institutions[0].display_name}
																					<svg
																						class="margin-left ml-2"
																						width="15"
																						height="15"
																						viewBox="0 0 15 15"
																						fill="none"
																						xmlns="http://www.w3.org/2000/svg"
																						><path
																							d="M8 2.75C8 2.47386 7.77614 2.25 7.5 2.25C7.22386 2.25 7 2.47386 7 2.75V7H2.75C2.47386 7 2.25 7.22386 2.25 7.5C2.25 7.77614 2.47386 8 2.75 8H7V12.25C7 12.5261 7.22386 12.75 7.5 12.75C7.77614 12.75 8 12.5261 8 12.25V8H12.25C12.5261 8 12.75 7.77614 12.75 7.5C12.75 7.22386 12.5261 7 12.25 7H8V2.75Z"
																							fill="currentColor"
																							fill-rule="evenodd"
																							clip-rule="evenodd"
																						></path></svg
																					>
																				</Button>
																			</Sheet.Close>
																		{:else}
																			<div></div>
																		{/if}
																	{/each}
																	<hr style="margin-top: .5rem; margin-bottom: .5rem" />
																	{#if work.referenced_works_count > 0}
																		<span style="font-weight: bold;"> Cites: </span>
																		<!-- <a
																			href="https://openalex.org/works?page=1&filter=cited_by:{work.id.substring(
																				'https://openalex.org/'.length
																			)}"
																			target="_blank"
																			rel="noreferrer noopener"
																			class="text-blue-500 hover:underline"
																		>
																			{work.referenced_works_count}
																		</a> <br /> -->
																		<Sheet.Close asChild let:builder>
																			<Button
																				builders={[builder]}
																				variant="ghost"
																				size="sm"
																				class="text text-base font-normal text-blue-500"
																				style="text-transform: capitalize"
																				on:click={() =>
																					setCitedBy(
																						work.id.substring('https://openalex.org/'.length)
																					)}
																				>{work.referenced_works_count}
																			</Button> <br />
																		</Sheet.Close>
																	{:else}
																		<span></span>
																	{/if}
																	{#if work.cited_by_count > 0}
																		<span style="font-weight: bold;"> Cited by: </span>
																		<!-- <a
																			href="https://openalex.org/works?page=1&filter=cites:{work.id.substring(
																				'https://openalex.org/'.length
																			)}"
																			target="_blank"
																			rel="noreferrer noopener"
																			class="text-blue-500 hover:underline"
																		>
																			{work.cited_by_count}
																		</a> <br /> -->
																		<Sheet.Close asChild let:builder>
																			<Button
																				builders={[builder]}
																				variant="ghost"
																				size="sm"
																				class="text text-base font-normal text-blue-500"
																				style="text-transform: capitalize"
																				on:click={() =>
																					setCites(
																						work.id.substring('https://openalex.org/'.length)
																					)}
																				>{work.cited_by_count}
																			</Button> <br />
																		</Sheet.Close>
																	{:else}
																		<span></span>
																	{/if}
																	{#if work.related_works.length > 0}
																		<span style="font-weight: bold;"> Related to: </span>
																		<!-- <a
																			href="https://openalex.org/works?page=1&filter=related_to:{work.id.substring(
																				'https://openalex.org/'.length
																			)}"
																			target="_blank"
																			rel="noreferrer noopener"
																			class="text-blue-500 hover:underline"
																		>
																			{work.related_works.length}
																		</a> <br /> -->
																		<Sheet.Close asChild let:builder>
																			<Button
																				builders={[builder]}
																				variant="ghost"
																				size="sm"
																				class="text text-base font-normal text-blue-500"
																				style="text-transform: capitalize"
																				on:click={() =>
																					setRelated(
																						work.id.substring('https://openalex.org/'.length)
																					)}
																				>{work.related_works.length}
																			</Button> <br />
																		</Sheet.Close>
																	{:else}
																		<span></span>
																	{/if}
																	<hr style="margin-top: .5rem; margin-bottom: .5rem" />
																	{#if work.primary_topic == null}
																		<div></div>
																	{:else}
																		<span style="font-weight: bold;"> Topic: </span>
																		<a
																			href={work.primary_topic.id}
																			target="_blank"
																			rel="noreferrer noopener"
																			class="text-blue-500 hover:underline"
																		>
																			{work.primary_topic.display_name}
																		</a> <br />
																		<span style="font-weight: bold;"> Subfield: </span>
																		<a
																			href={work.primary_topic.subfield.id}
																			target="_blank"
																			rel="noreferrer noopener"
																			class="text-blue-500 hover:underline"
																		>
																			{work.primary_topic.subfield.display_name}
																		</a> <br />
																		<span style="font-weight: bold;"> Field: </span>
																		<a
																			href={work.primary_topic.field.id}
																			target="_blank"
																			rel="noreferrer noopener"
																			class="text-blue-500 hover:underline"
																		>
																			{work.primary_topic.field.display_name}
																		</a> <br />
																		<span style="font-weight: bold;"> Domain: </span>
																		<a
																			href={work.primary_topic.domain.id}
																			target="_blank"
																			rel="noreferrer noopener"
																			class="text-blue-500 hover:underline"
																		>
																			{work.primary_topic.domain.display_name}
																		</a> <br />
																	{/if}
																	{#if work.sustainable_development_goals == undefined || work.sustainable_development_goals.length == 0}
																		<div></div>
																	{:else}
																		<span style="font-weight: bold;">
																			Sustainable Development Goal:
																		</span>
																		<a
																			href={work.sustainable_development_goals[0].id}
																			target="_blank"
																			rel="noreferrer noopener"
																			class="text-blue-500 hover:underline"
																		>
																			{work.sustainable_development_goals[0].display_name}
																		</a> <br />
																	{/if}
																	<hr style="margin-top: .5rem; margin-bottom: .5rem" />
																	<span style="font-weight: bold;"> Open Access Status: </span>
																	<Sheet.Close asChild let:builder>
																		<!-- style="text-transform: capitalize" -->
																		<Button
																			builders={[builder]}
																			variant="ghost"
																			size="sm"
																			class="text text-base font-normal text-blue-500"
																			on:click={() => setStatus(work.open_access.oa_status)}
																			>{work.open_access.oa_status}
																			<svg
																				class="margin-left ml-2"
																				width="15"
																				height="15"
																				viewBox="0 0 15 15"
																				fill="none"
																				xmlns="http://www.w3.org/2000/svg"
																				><path
																					d="M8 2.75C8 2.47386 7.77614 2.25 7.5 2.25C7.22386 2.25 7 2.47386 7 2.75V7H2.75C2.47386 7 2.25 7.22386 2.25 7.5C2.25 7.77614 2.47386 8 2.75 8H7V12.25C7 12.5261 7.22386 12.75 7.5 12.75C7.77614 12.75 8 12.5261 8 12.25V8H12.25C12.5261 8 12.75 7.77614 12.75 7.5C12.75 7.22386 12.5261 7 12.25 7H8V2.75Z"
																					fill="currentColor"
																					fill-rule="evenodd"
																					clip-rule="evenodd"
																				></path></svg
																			>
																		</Button>
																	</Sheet.Close>
																	<br />
																</div>
																<Sheet.Footer>
																	<Sheet.Close asChild let:builder>
																		<!-- on:click={() => setWorkType('"' + work.type + '"')} -->
																		<Button
																			builders={[builder]}
																			variant="secondary"
																			on:click={() => setWorkType(work.type)}
																			>Filter by type - {work.type}
																		</Button>
																	</Sheet.Close>
																</Sheet.Footer>
															</div></Sheet.Content
														>
													</Sheet.Root>
												</div>
											</Table.Cell>
										</Table.Row>
									{/each}
								</Table.Body>
							</Table.Root>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>
