<script lang="ts">
	import { onMount } from 'svelte';
	import { colors } from './colors';

	interface Size {
		x: number;
		y: number;
	}

	var size: Size = { x: 10, y: 10 };
	var data: number[][] = createDataArray(size);
	var selected: number = 31;
	var isMouseDown: boolean;
	var coordinate = { x: 0, y: 0 };

	function createDataArray(size: Size): number[][] {
		const newData: number[][] = [];
		for (let i = 0; i < size.y; i++) {
			const row: number[] = [];
			for (let j = 0; j < size.x; j++) {
				row.push(31); // white
			}
			newData.push(row);
		}
		return newData;
	}

	function updateDataArray(newSize: Size): void {
		const newData: number[][] = createDataArray(newSize);
		const rowsToCopy = Math.min(newSize.y, data.length);
		const colsToCopy = Math.min(newSize.x, data[0]?.length || 0);

		for (let i = 0; i < rowsToCopy; i++) {
			for (let j = 0; j < colsToCopy; j++) {
				newData[i][j] = data[i][j];
			}
		}

		data = newData;
	}

	function shiftDataArray(x: number, y: number): void {
		const newData: number[][] = createDataArray(size);

		for (let i = 0; i < size.y; i++) {
			for (let j = 0; j < size.x; j++) {
				const newRow = i + y;
				const newCol = j + x;

				// Check if the new row and column are within the valid range
				if (newRow >= 0 && newRow < size.y && newCol >= 0 && newCol < size.x) {
					newData[newRow][newCol] = data[i][j];
				}
			}
		}

		data = newData;
	}

	$: {
		updateDataArray(size);
		data = data;
	}

	// Function to encode data and size into a query string
	function encodeDataAndCoordinate(): string {
		const encodedData = encodeURIComponent(JSON.stringify(data));
		const encodedCoordinate = encodeURIComponent(JSON.stringify(coordinate));
		const encodedSize = encodeURIComponent(JSON.stringify(size));
		return `${location.origin}?data=${encodedData}&coordinate=${encodedCoordinate}&size=${encodedSize}`;
	}

	// Function to decode data from the query parameters
	function decodeData(queryParams: string) {
		const urlParams = new URLSearchParams(queryParams);
		const decodedData = urlParams.get('data');
		const decodedCoordinate = urlParams.get('coordinate');
		const decodedSize = urlParams.get('size');

		if (decodedData && decodedCoordinate && decodedSize) {
			try {
				data = JSON.parse(decodeURIComponent(decodedData));
				coordinate = JSON.parse(decodeURIComponent(decodedCoordinate));
				size = JSON.parse(decodeURIComponent(decodedSize));
			} catch (error) {
				console.error('Error decoding data:', error);
			}
		}
	}

	async function copyToClipboard(text: string) {
		try {
			await navigator.clipboard.writeText(text);
			console.log('Text copied to clipboard:', text);
		} catch (error) {
			console.error('Error copying to clipboard:', error);
		}
	}

	// Listen for URL changes and decode data
	onMount(() => {
		const queryParams = location.search;
		decodeData(queryParams);
	});
</script>

<!-- svelte-ignore a11y-no-static-element-interactions -->
<div
	class="flex items-center justify-center min-h-screen"
	on:mousedown={() => {
		isMouseDown = true;
	}}
	on:mouseup={() => {
		isMouseDown = false;
	}}
>
	<!-- Your data will be populated here -->
	<div class="flex flex-col">
		<div class="h-20 flex items-center justify-center flex-row gap-3">
			<svg
				xmlns="http://www.w3.org/2000/svg"
				fill="none"
				viewBox="0 0 24 24"
				stroke-width="1.5"
				stroke="currentColor"
				class="w-6 h-6"
			>
				<path
					stroke-linecap="round"
					stroke-linejoin="round"
					d="M15 10.5a3 3 0 11-6 0 3 3 0 016 0z"
				/>
				<path
					stroke-linecap="round"
					stroke-linejoin="round"
					d="M19.5 10.5c0 7.142-7.5 11.25-7.5 11.25S4.5 17.642 4.5 10.5a7.5 7.5 0 1115 0z"
				/>
			</svg>

			<p>(</p>
			<input class="h-10 w-12 font text-center" type="number" id="x" bind:value={coordinate.x} />
			<p>,</p>
			<input class="h-10 w-12 font text-center" type="number" id="y" bind:value={coordinate.y} />
			<p>)</p>

			<svg
				xmlns="http://www.w3.org/2000/svg"
				fill="none"
				viewBox="0 0 24 24"
				stroke-width="1.5"
				stroke="currentColor"
				class="w-6 h-6"
			>
				<path
					stroke-linecap="round"
					stroke-linejoin="round"
					d="M3.75 3.75v4.5m0-4.5h4.5m-4.5 0L9 9M3.75 20.25v-4.5m0 4.5h4.5m-4.5 0L9 15M20.25 3.75h-4.5m4.5 0v4.5m0-4.5L15 9m5.25 11.25h-4.5m4.5 0v-4.5m0 4.5L15 15"
				/>
			</svg>

			<p>(</p>
			<input class="h-10 w-12 font text-center" type="number" id="x" min="1" bind:value={size.x} />
			<p>,</p>
			<input class="h-10 w-12 font text-center" type="number" id="y" min="1" bind:value={size.y} />
			<p>)</p>

			<button class="h-10 w-10 font text-center" on:click={() => shiftDataArray(0, -1)}
				><svg
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					stroke-width="1.5"
					stroke="currentColor"
					class="w-6 h-6"
				>
					<path stroke-linecap="round" stroke-linejoin="round" d="M4.5 15.75l7.5-7.5 7.5 7.5" />
				</svg>
			</button>
			<button class="h-10 w-10 font text-center" on:click={() => shiftDataArray(0, 1)}
				><svg
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					stroke-width="1.5"
					stroke="currentColor"
					class="w-6 h-6"
				>
					<path stroke-linecap="round" stroke-linejoin="round" d="M19.5 8.25l-7.5 7.5-7.5-7.5" />
				</svg>
			</button>
			<button class="h-10 w-10 font text-center" on:click={() => shiftDataArray(-1, 0)}
				><svg
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					stroke-width="1.5"
					stroke="currentColor"
					class="w-6 h-6"
				>
					<path stroke-linecap="round" stroke-linejoin="round" d="M15.75 19.5L8.25 12l7.5-7.5" />
				</svg>
			</button>
			<button class="h-10 w-10 font text-center" on:click={() => shiftDataArray(1, 0)}
				><svg
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					stroke-width="1.5"
					stroke="currentColor"
					class="w-6 h-6"
				>
					<path stroke-linecap="round" stroke-linejoin="round" d="M8.25 4.5l7.5 7.5-7.5 7.5" />
				</svg>
			</button>
			<button
				class="h-10 w-10 font text-center"
				on:click={() => {
					data = createDataArray(size);
				}}
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					stroke-width="1.5"
					stroke="currentColor"
					class="w-6 h-6"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0"
					/>
				</svg>
			</button>

			<button
				class="h-10 w-10 font text-center"
				on:click={() => {
					copyToClipboard(encodeDataAndCoordinate());
				}}
			>
				<svg
					xmlns="http://www.w3.org/2000/svg"
					fill="none"
					viewBox="0 0 24 24"
					stroke-width="1.5"
					stroke="currentColor"
					class="w-6 h-6"
				>
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						d="M7.217 10.907a2.25 2.25 0 100 2.186m0-2.186c.18.324.283.696.283 1.093s-.103.77-.283 1.093m0-2.186l9.566-5.314m-9.566 7.5l9.566 5.314m0 0a2.25 2.25 0 103.935 2.186 2.25 2.25 0 00-3.935-2.186zm0-12.814a2.25 2.25 0 103.933-2.185 2.25 2.25 0 00-3.933 2.185z"
					/>
				</svg>
			</button>
		</div>

		<!-- Wrap the rows in a container with flex flex-col classes -->
		<div class="flex items-center justify-center flex-row">
			<p class="h-10 w-10 flex items-center justify-center bg-white" />
			{#each { length: size.x } as _, j}
				<p
					class="border-l border-white h-10 w-10 min-h-[10] flex items-center justify-center bg-stone-200"
				>
					{j + coordinate.x}
				</p>
			{/each}
		</div>

		{#each data as row, i}
			<div class="flex items-center justify-center">
				<p class="border-t border-white h-10 w-10 flex items-center justify-center bg-stone-200">
					{i + coordinate.y}
				</p>
				{#each row as item, j}
					<!-- svelte-ignore a11y-mouse-events-have-key-events -->
					<button
						class="border h-10 w-10 flex items-center justify-center"
						style="background-color: {colors[item].hex};"
						on:mousedown={() => {
							data[i][j] = selected;
							data = data;
						}}
						on:mouseover={() => {
							if (isMouseDown) {
								data[i][j] = selected;
								data = data;
							}
						}}
					/>
				{/each}
			</div>
		{/each}

		<div class="mt-auto h-40 p-5 flex items-center justify-center">
			{#each colors as color, i}
				<button
					on:click={() => {
						selected = i;
						console.log(selected);
					}}
					class={'rounded-md h-10 w-10 ' + (selected == i ? 'border-2 border-black' : 'border')}
					style="background-color: {color.hex};"
				/>
			{/each}
		</div>
	</div>
</div>

<style>
	p,
	.font {
		font-family: 'Jetbrains Mono';
	}
	div {
		user-select: none;
	}
</style>
