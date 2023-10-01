<script>	
	let data = []
	let mass = null
	let density = 0.792
	let copyMessage = ''

	$: volume = (mass / density).toFixed(4)

	function submitData() {
		data = [...data, { mass, volume }]
		mass = ''
		window.scrollTo(0, document.documentElement.scrollHeight)
	}

	function handleEdit(i, e) {
		const newMass = parseFloat(e.target.textContent)

		if (!isNaN(newMass)) {
			const newVolume = (newMass / density).toFixed(4)
			e.target.nextElementSibling.textContent = newVolume
		} else {
			e.target.nextElementSibling.textContent = 0
		}
	}

	function handleKeyDown(e) {
		if (e.key == 'Enter') {
			e.preventDefault()
			e.target.blur()
		}
	}

	function deleteRow(i) {
		const confirmDelete = confirm(`Are you sure you want to delete the following row?\n\nMass: ${data[i].mass}\nVolume: ${data[i].volume}`)
		if (confirmDelete) {
			data = [...data.slice(0, i), ...data.slice(i + 1)]
		}
	}

	function finalizeEdit(i, e) {
		const newMass = parseFloat(e.target.textContent);
		if (!isNaN(newMass)) {
			const newVolume = (newMass / density).toFixed(4)
			data[i] = { mass: newMass, volume: newVolume }
		} else {
			data[i] = { mass: 0, volume: 0 }
		}
	}

	async function copyVolumes() {
		const volumes = data.map(sample => sample.volume).join('\n')

		try {
			await navigator.clipboard.writeText(volumes)
			copyMessage = '\u2713 Extraction Volumes Copied'
			setTimeout(() => {
				copyMessage = ''
			}, 5000)
		} catch (e) {
			copyMessage = 'Failed to copy volume data: ' + e
		}
	}
</script>

<main>
	<div class="column-headings">
		<h5 class="heading">Mass (g)</h5>
		<h5 class="heading">Extraction Volume (mL)</h5>
	</div>
	<div class="sample-list">
		{#each data as row, i}
			<div class="data-row">
				<span 
					class="mass" 
					contenteditable="true" 
					on:input={e => handleEdit(i, e)}
					on:blur={e => finalizeEdit(i, e)}
					on:keydown={handleKeyDown}>
					{row.mass}
				</span>
				<!-- svelte-ignore a11y-click-events-have-key-events -->
				<span class="volume" on:click={copyVolumes}>
					{row.volume}
				</span>
				<button class="delete" on:click={() => deleteRow(i)}>-</button>
			</div>
		{/each}
	</div>

	<div class="calculator">
		<div>
			<form on:submit|preventDefault={submitData}>
				<input type="number" step="0.0001" bind:value={mass} />
			</form>
			<span class="volume">{volume}</span>
		</div>
	</div>
	
	<div class="copy-message">{copyMessage}</div>
</main>

<style>
	main {
		display: flex;
		flex-direction: column;
		padding: 120px;
	}

	.calculator {
		margin-bottom: 3rem;
	}

	.sample-list > .data-row:last-child {
		margin-bottom: 2rem;
	}

	h5 {
		color: #777;
		display: inline-block;
		font-weight: 400;
		font-size: 12px;
		margin-bottom: 16px;
		padding: 3px;
		width: 180px;
	}

	form, span {
		display: inline-block;
	}

	input, span {
		border-radius: 5px;
		line-height: 40px;
		font-size: 14px;
		padding: 0 16px;
		width: 180px;
	}

	input:focus {
		outline: none;
	}

	span, input {
		color: rgba(0, 0, 0, 0.5);
	}

	input {
		background: none;
		border: 1px solid #eee;
		color: #566;
		font-size: 14px;
	}

	.mass, .volume {
		margin-bottom: 4px;
	}

	.sample-list .volume {
		cursor: pointer;
		background: #e8eee8;
	}

	.volume {
		background: #f4f4f4;
	}

	.mass {
		background: #f4f4f4;
		margin-right: 4.5px;
	}

	.mass:focus {
		outline: none;
		background: #fafafa;
	}

	.data-row {
		display: flex;
		align-items: flex-start;
		position: relative;
	}

	.copy-message {
		color: #777;
		font-size: 14px;
		position: fixed;
		right: 24px;
		top: 20px;
	}

	.delete {
		cursor: pointer;
		border: none;
		background: none;
		color: #888;
		padding: 1rem;
		position: absolute;
		right: -44px;
	}

	.delete:focus {
		background: none;
		border: none;
	}
</style>
