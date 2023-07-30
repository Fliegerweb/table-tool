<script lang="ts">
	import { writeText, readText } from '@tauri-apps/api/clipboard';

	const INSERT_TEXT_ERROR = 'Please insert the text to be converted into the upper text box';
	const EMPTY_TEXT_ERROR = 'Inserted text is empty';

	const TABLE_PREFIX = '<table>\n<tbody>\n';
	const TABLE_POSTFIX = '</tbody>\n</table>';

	const TEXTAREA_ROWS = 5;

	let inputData: string;
	let outputData = '';

	let previewActive = false;
	let isErrorState = false;
	let copySuccess = false;

	$: if (!inputData) {
		outputData = INSERT_TEXT_ERROR;
		isErrorState = true;
	} else if (/^\s*$/.test(outputData)) {
		outputData = EMPTY_TEXT_ERROR;
		isErrorState = true;
	} else {
		try {
			// Try to convert data to table
			outputData = createHtmlTable(inputData);
			isErrorState = false;
		} catch (err) {
			outputData = (err as Error).toString();
			isErrorState = true;
		}
	}

	$: if (isErrorState) previewActive = false;

	function createHtmlTable(data: string): string {
		const lines = data.split(/\r?\n|\r|\n/g);

		const tableRows: Array<string> = [];

		lines.forEach((line, idx) => {
			// Check if line is valid
			const matchColonResult = Array.from(line.matchAll(/:/g));
			if (matchColonResult.length === 0 && line.trim().length !== 0) {
				throw new Error(
					`Found a non-empty line without any : . Please check your input.\n\nAffected line: ${
						idx + 1
					}\nContent: ${line}`
				);
			} else if (matchColonResult.length === 0) {
				// line is empty
				return;
			} else if (matchColonResult.length > 1) {
				throw new Error(
					`Found a line with multiple : . Please check your input.\n\nAffected line: ${
						idx + 1
					}\nContent: ${line}`
				);
			}

			let parts = line.split(':');

			parts = parts.map((part) => {
				// Check if parts are valid and trim them
				const trimmed = part.trim();

				if (trimmed.length === 0)
					throw new Error(
						`Splitting at : caused an empty string part. Please check your input.\n\nAffected line: ${
							idx + 1
						}\nContent: ${line}`
					);

				return trimmed;
			});

			tableRows.push(`<tr>\n<td>${parts[0]}</td>\n<td>${parts[1]}</td>\n</tr>\n`);
		});

		// Check if table is empty
		if (tableRows.length === 0)
			throw new Error(`Conversion resulted in empty table. Please check your input.`);

		return TABLE_PREFIX + tableRows.join('') + TABLE_POSTFIX;
	}

	async function insertClipboard() {
		if (inputData && inputData.length !== 0) return; // Do not do anything if there already is text

		const clipboardData = await readText();

		if (clipboardData && clipboardData.trim().length !== 0) {
			inputData = clipboardData;
		}
	}

	async function copyToClipboard() {
		await writeText(outputData);

		showSuccessToast();
	}

	function showSuccessToast() {
		if (copySuccess) return;

		copySuccess = true;

		setTimeout(() => {
			copySuccess = false;
		}, 2000);
	}
</script>

<div class="flex w-full h-full flex-col">
	<div class="flex-1">
		<textarea
			class="textarea textarea-primary"
			on:click={() => insertClipboard()}
			bind:value={inputData}
			placeholder="paste text to convert"
			rows={TEXTAREA_ROWS}
		/>
	</div>
	<div class="flex-1">
		<div class="tabs justify-center mt-8 mb-4">
			<!-- svelte-ignore a11y-missing-attribute -->
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<!-- svelte-ignore a11y-no-static-element-interactions -->
			<a
				class="tab tab-bordered transition-colors"
				class:tab-active={!previewActive}
				on:click={() => (previewActive = false)}>HTML Output</a
			>
			<!-- svelte-ignore a11y-missing-attribute -->
			<!-- svelte-ignore a11y-click-events-have-key-events -->
			<!-- svelte-ignore a11y-no-static-element-interactions -->
			<a
				class="tab tab-bordered transition-colors"
				class:tab-active={previewActive}
				class:tab-disabled={isErrorState}
				on:click={() => {
					if (!isErrorState) previewActive = true;
				}}>Preview</a
			>
		</div>

		{#if !previewActive}
			<textarea
				on:click={() => {
					if (!isErrorState) copyToClipboard();
				}}
				class="textarea"
				class:textarea-error={isErrorState}
				class:textarea-success={!isErrorState}
				value={outputData}
				rows={TEXTAREA_ROWS}
				spellcheck={false}
			/>
			<div class="flex justify-center">
				<button
					on:click={() => {
						if (!isErrorState) copyToClipboard();
					}}
					class="btn m-4"
					class:btn-error={isErrorState}
					class:btn-accent={!isErrorState}
				>
					Copy to clipboard
				</button>
			</div>
		{:else}
			<div class="prose xl:prose-xl mb-4">
				<h3>Sample title</h3>
				<p>
					Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor
					invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua.
				</p>

				{@html outputData}

				<p>
					Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor
					invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua.
				</p>
			</div>
		{/if}
	</div>
</div>

{#if copySuccess}
	<div class="toast toast-top toast-center">
		<div class="alert alert-success">
			<span>Successfully copied to clipboard</span>
		</div>
	</div>
{/if}

<style>
	textarea {
		width: 100%;
	}
</style>
