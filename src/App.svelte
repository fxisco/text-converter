<style>
	.letter {
		cursor: pointer;
		border: 1px solid black;
		display: inline-flex;
		padding: 4px;
		flex-direction: column;
		height: 50px;
		justify-content: center;
		margin-left: 2px;
		margin-bottom: 30px;
		text-align: center;
		width: 50px;
		white-space: pre-wrap;
	}

	.result-title {
		display: inline-block;
	}

	.letter:hover {
		background-color: #add8e6;
	}

	.letter:active {
		background-color: #00FFFF;
	}

	.text-result > .letter:last-child {
		border-right: 1px solid black;
	}

	.text-result {
		position: relative;
	}

	.original,
	.transformation,
	.copy-text {
		pointer-events: none;
	}

	.copy-text {
		bottom: 0;
		position: absolute;
	}

	.original {
		height: 18px;
	}

	.title {
    display: inline-block;
  }

	.selector {
		display: inline-block;
	}

	.text-area {
		overflow: hidden;
		padding:10px;
		resize: none;
		height: 100%;
		width: 100%;
	}

	.text-area-container {
		position: relative;
	}
</style>
<script>
	import { fade } from 'svelte-transitions';
	const DEFAULT_SELECTION = 'ascii',
		converters = {
			'ascii': getASCIICode,
			'hex': getHEXCode,
			'html': getHTMLCode,
			'oct': getOCTCode,
		},
		COPY_TO_CLIPBOARD_TIMEOUT = 500,
		COPY_TEXT_OFFSET = 5;

	let input = '',
			copied = false,
			top = 0,
			left = 0,
			selection = [DEFAULT_SELECTION],
			timer;

	function getSelectionCode(letter) {
		return converters[selection](letter);
	}

	function getASCIICode(letter) {
		return letter.charCodeAt(0);
	}

	function getHEXCode(letter) {
		return letter.charCodeAt(0).toString(16);
	}

	function getOCTCode(letter) {
		return letter.charCodeAt(0).toString(8);
	}

	function getHTMLCode(letter) {
		return '&#'+ letter.charCodeAt(0) + ';'
	}

	function handleSelection() {
		const previousValue = input;
		input = '';
		input = previousValue;
	}

	function handleClick(value, event) {
		const item = event.target,
					parent = event.target.parentElement,
					itemCoords = item.getBoundingClientRect(),
					parentCoords = parent.getBoundingClientRect();

		left = itemCoords.left - COPY_TEXT_OFFSET;
		top = itemCoords.bottom - parentCoords.top + COPY_TEXT_OFFSET;

		copyText(value);
	}

	function copyText(value) {
		copied = true;

		copyToClipboard(value);
		clearTimeout(timer);
		timer = setTimeout(() => {
			copied = false;
		}, COPY_TO_CLIPBOARD_TIMEOUT);
	}

	function copyToClipboard(str) {
		const el = document.createElement('textarea');

		el.value = str;

		document.body.appendChild(el);

		el.select();

		document.execCommand('copy');
		document.body.removeChild(el);
	};

	function resize(event) {
		const { target } = event;

		setTimeout(function(){
			target.style.cssText = 'height:auto; padding:10px';
			target.style.cssText = 'height:' + target.scrollHeight + 'px';
		},0);
	}
</script>
<div class="body">
	<div>
		<h1 class="title">Conver text to: </h1>
		<select class="selector" bind:value={selection} on:change={handleSelection}>
			<option value="ascii">ASCII</option>
			<option value="hex">HEX</option>
			<option value="oct">OCT</option>
		</select>
	</div>
	<div>
		<div class="text-area-container">
			<textarea rows="1" class="text-area" bind:value={input} on:keydown={resize}></textarea>
		</div>
		{#if input}
			<div transition:fade>
				<h2 class="result-title">Result:</h2>
				<div class="text-result">
					{#if copied}
						<span class="copy-text" style="top:{top}px; left:{left}px">Copied</span>
					{/if}
					{#each input.split('') as letter }
						<div
						  class="letter"
							on:click={handleClick.bind(this, getSelectionCode(letter))}
						>
							<div class="original">
								{letter}
							</div>
							<div class="transformation">{getSelectionCode(letter)}</div>
						</div>
						{#if getASCIICode(letter) === 10}
							<div></div>
						{/if}
					{/each}
				</div>
			</div>
		{/if}
	</div>
</div>
