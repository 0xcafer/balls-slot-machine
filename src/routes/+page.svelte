<script>
	import { browser } from '$app/environment';
	import { onMount } from 'svelte';
	let blocksForPerRow = 3;
	let rowsCount = 3;
	let rows;
	const items = [
		'🐶',
		'🐱',
		'🐭',
		'🐹',
		'🐰',
		'🦊',
		'🐻',
		'🐼',
		'🐻‍❄️',
		// '🐨',
		// '🐯',
		// '🦁',
		// '🐮',
		// '🐷'
		// '🍭',
		// '❌',
		// '⛄️',
		// '🦄',
		// '🍌',
		// '💩',
		// '👻',
		// '😻',
		// '💵',
		// '🤡',
		// '🦖',
		// '🍎',
		// '😂',
		// '🖕'
	];

	let tmpTour = new Array(blocksForPerRow * rowsCount);

	const init = (firstInit = true, groups = 1, duration = 1) => {
		rows = document.querySelectorAll('.block');

		for (const [blockIndex, block] of rows.entries()) {
			if (firstInit) {
				block.dataset.spinned = '0';
			}

			const boxes = block.querySelector('.boxes');

			const boxesClone = boxes.cloneNode(false);
			const pool = [];
			const arr = [];
			for (let n = 0; n < (groups > 0 ? groups : 1); n++) {
				arr.push(...items);
			}
			pool.push(...shuffle(arr));

			if (!firstInit) {
				boxesClone.addEventListener(
					'transitionstart',
					function () {
						block.dataset.spinned = parseInt(block.dataset.spinned) + 1;
						this.querySelectorAll('.box').forEach((box) => {
							box.style.filter = 'blur(1px)';
						});
					},
					{ once: true }
				);

				boxesClone.addEventListener(
					'transitionend',
					function () {
						this.querySelectorAll('.box').forEach((box, index) => {
							box.style.filter = 'blur(0)';
							if (index > 0) this.removeChild(box);
						});
					},
					{ once: true }
				);
			}

			let poolLenght = pool.length - 1;

			for (let i = poolLenght; i >= 0; i--) {
				const box = document.createElement('div');
				box.classList.add('box');
				box.style.width = block.clientWidth + 'px';
				box.style.height = block.clientHeight + 'px';

				if (i == 0 && tmpTour[blockIndex] && tmpTour[blockIndex][0]) {
					box.textContent = tmpTour[blockIndex][0];
				} else {
					if (i == 0) {
						if (tmpTour[blockIndex] && tmpTour[blockIndex][i]) {
							tmpTour[blockIndex][i] = pool[i];
						} else {
							tmpTour[blockIndex] = [];
							tmpTour[blockIndex][i] = pool[i];
						}
					}
					box.textContent = pool[i];
				}

				boxesClone.appendChild(box);
			}

			boxesClone.style.transitionDuration = `${duration > 0 ? duration : 1}s`;
			boxesClone.style.transform = `translateY(-${block.clientHeight * (pool.length - 1)}px)`;
			block.replaceChild(boxesClone, boxes);
		}
	};

	const spin = async () => {
		init(false, 1, 2);

		for (const block of rows) {
			const boxes = block.querySelector('.boxes');
			const duration = parseInt(boxes.style.transitionDuration);
			boxes.style.transform = 'translateY(0)';
			await new Promise((resolve) => setTimeout(resolve, duration * 100));
		}
	};

	const shuffle = ([...arr]) => {
		let m = arr.length;
		while (m) {
			const i = Math.floor(Math.random() * m--);
			[arr[m], arr[i]] = [arr[i], arr[m]];
		}
		return arr;
	};

	if (browser) {
		onMount(() => {
			init();
		});
	}
</script>

<div class="w-full h-full bg-teal-800 flex flex-col justify-center items-center text-white">
	{#each new Array(rowsCount) as row}
		<div class="rows flex">
			{#each new Array(blocksForPerRow) as block}
				<div class="block bg-white w-[100px] h-[110px] overflow-hidden rounded-md m-1">
					<div class="boxes" />
				</div>
			{/each}
		</div>
	{/each}

	<div class="flex flex-row gap-x-8 mt-8 w-3/12">
		<button
			class="border rounded-md p-3 mx-auto active:bg-white active:text-teal-800 lg:hover:bg-white lg:hover:text-teal-800 w-full md:w-6/12"
			on:click={() => spin()}>Spin</button
		>
	</div>
</div>
