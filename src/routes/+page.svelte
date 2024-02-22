<script>
	import { onMount } from "svelte";

	let program = "++++++++++[>+++++++>++++++++++>+++>+<<<<-]>++.>+.+++++++..+++.>++.<<+++++++++++++++.>.+++.------.--------.>+.>.";
	let output = "";
	/** @type {string[]} */
	let input = [];
	let i = 0;
	let d = 0;
	let delay = 0;

	let running = false;
	let instant = false;

	let memory = new Array(30000).fill(0);

	/** @type {HTMLTextAreaElement} */
	let program_source;

	onMount(() => {
		program_source = /** @type {HTMLTextAreaElement} */ (document.getElementById("program-source"));
	});;

	/** @param {boolean} queueNextStep */
	const step = (queueNextStep = false) => {
		if (i >= program.length) {
			running = false;
			return;
		}
		if (!running) {
			program_source.focus();
			program_source.setSelectionRange(i - 1, i);
			return;
		}
		if (instant && queueNextStep) {
			return;
		}
		// select program_source and move cursor to i
		program_source.focus();
		program_source.setSelectionRange(i, i + 1);

		switch (program[i]) {
			case "+":
				memory[d]++;
				break;
			case "-":
				memory[d]--;
				break;
			case ">":
				d++;
				break;
			case "<":
				d--;
				break;
			case "[":
				if (memory[d] === 0) {
					let count = 1;
					while (count > 0) {
						i++;
						if (program[i] === "[") count++;
						if (program[i] === "]") count--;
					}
				}
				break;
			case "]":
				if (memory[d] !== 0) {
					let count = 1;
					while (count > 0) {
						i--;
						if (program[i] === "[") count--;
						if (program[i] === "]") count++;
					}
				}
				break;
			case ".":
				if (memory[d] == 10) {
					output += `
`;
				} else {
					output += String.fromCharCode(memory[d])
				}
				break;
			case ",":
				if (input.length > 0) {
					memory[d] = input.shift().charCodeAt(0);
					input = input;
				} else {
					const char = prompt("Input a character");
					if (!char) {
						memory[d] = 0;
					} else {
						input.push(...char.split(""));
						memory[d] = input.shift().charCodeAt(0);
						input = input;
					}
				}
				break;
		}
		i++;
		if (queueNextStep && i < program.length && running) {
			setTimeout(() => {
				step(true);
			}, delay);
		}
	}

	/** @param {boolean} stepThrough */
	const start = (stepThrough = true) => {
		reset();
		running = true;
		
		if (stepThrough) {
			instant = false;
			step(true);
		} else {
			instant = true;
			while (i < program.length) {
				step(false);
			}
		}
	}

	const resume = () => {
		running = true;
		step(true);
	}

	const reset = () => {
		i = 0;
		d = 0;
		memory = memory.fill(0);
		output = "";
		program_source.focus();
		program_source.setSelectionRange(0, 0);
	}
</script>
<div class="content">

Program:
<textarea id="program-source" cols="30" rows="10" bind:value={program}></textarea>
Queued Input:
<span class="queued-input">{input.join("")}</span>
Output:
<textarea id="output-source" cols="30" rows="10">{output}</textarea>

<div class="controls">
	<div class="button-controls">
		<!-- Buttons: Start, Pause/Resume, Step, Reset, Run Instantly -->
		<button on:click={() => start(true)}>Start</button>
		<button on:click={() => {running = false; instant = false; reset()}}>Stop</button>
		<button on:click={() => running ? (running = false) : (resume())}>{running ? "Pause" : "Resume"}</button>
		<button on:click={() => step(false)}>Step ({i})</button>
		<button on:click={() => reset()}>Reset</button>
		<button on:click={() => start(false)}>Run Instantly</button>
	</div>
	<div class="other-controls">
		<div>
			<label for="delay">Delay ({delay}ms):</label>
			<input type="range" min="0" max="1000" bind:value={delay} />
		</div>
	</div>
</div>

</div>
<style>
	:global(*) {
		text-align: center;
		margin: auto;
	}

	:global(.caret-block) {
		background-color: black;
	}

	.content {
		margin: 10px;
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 10px;
	}

	.controls {
		display: flex;
		gap: 10px;
		flex-direction: column;
	}

	.other-controls > div {
		display: flex;
		gap: 10px;
		flex-direction: column;
	}

	textarea {
		text-align: left;
	}

	.queued-input {
		background-color: lightgray;
		padding: 5px;
		border-radius: 5px;
	}
</style>