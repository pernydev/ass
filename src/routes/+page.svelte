<script lang="ts">
	import EdDownstairs from '$lib/components/intercity/Ed/Downstairs.svelte';
	import EdUpstairs from '$lib/components/intercity/Ed/Upstairs.svelte';
	import EdfsDownstairs from '$lib/components/intercity/Edfs/Downstairs.svelte';
	import { seatData } from '$lib/seatdata';
	import { onMount } from 'svelte';

	let mousecoords = [0, 0];
	let lastElement: Element;
	let lastTouchTime = 0;
	let voice: SpeechSynthesisVoice | undefined;
	let voices: SpeechSynthesisVoice[] = [];

	let floor = 0;

	let landscapewarning = false;

	function handleMouseMove(event: MouseEvent) {
		mousecoords = [event.clientX, event.clientY];
		identify();
	}

	function handleFingerMove(event: TouchEvent) {
		console.log('touchmove');
		mousecoords = [event.touches[0].clientX, event.touches[0].clientY];
		identify();
	}

	function onClick(event: TouchEvent) {
		if (event.touches.length === 3) {
			floor = floor === 0 ? 1 : 0;
		}

		if (event.touches.length === 1) {
			const touch = event.touches[0];
			const button = document
				.elementFromPoint(touch.clientX, touch.clientY)
				?.closest('[data-button]');
			if (!button) return;

			switch (button.getAttribute('data-button')) {
				case 'left':
					switchwagon('previous');
					break;
				case 'right':
					switchwagon('next');
					break;
				case 'center':
					switchfloor();
					break;
			}
		}
	}

	onMount(() => {
		window.addEventListener('mousemove', handleMouseMove);
		window.addEventListener('touchmove', handleFingerMove);
		window.addEventListener('touchstart', onClick);

		return () => {
			window.removeEventListener('mousemove', handleMouseMove);
			window.removeEventListener('touchmove', handleFingerMove);
			window.removeEventListener('touchstart', onClick);
		};
	});

	let wagon = 1;

	let configuration: Record<number, string> = {
		1: 'Ed',
		2: 'Edfs'
	};

	let lang = 'en-US';

	const i18n: Record<string, Record<string, string>> = {
		'en-US': {
			wagon: 'Coach',
			seat: 'Seat',
			table: 'Table',
			'luggage-storage': 'Luggage storage',
            'ski-storage': 'Ski storage',

			'private-compartment': 'Private compartment',
            'family-compartment': 'Family compartment',
			toilet: 'Toilet',
			'telephone-booth': 'Telephone booth',
			stairs: 'Stairs',
			door: 'Door',

            wheelchair: 'Wheelchair',

			'big-private-compartment': 'Big private compartment',
			'small-private-compartment': 'Small private compartment',
			window: 'Window',
			asile: 'Asile',
			'no-recline': "Doesn't recline",
			'next-to-table': 'Next to table'
		}
	};

	function say(text: string) {
		window.speechSynthesis.cancel();
		const utterance = new SpeechSynthesisUtterance(text);
		if (voice) utterance.voice = voice;
		window.speechSynthesis.speak(utterance);
	}

	const handlers: Record<string, (element: Element) => void> = {
		seat: (element: Element) => {
			let seat = element.id.split('seat_')[1];
			if (seat.endsWith('_shape')) {
				seat = seat.split('_shape')[0];
			}

			const features = seatData[configuration[wagon]][parseInt(seat)].map(
				(feature) => i18n[lang][feature]
			);
			say(`${i18n[lang].seat} ${seat} ${features.join(', ')}`);

			navigator.vibrate(20);
		},
		table: (element: Element) => {
			say(i18n[lang].table);
			navigator.vibrate(100);
		},
		'luggage-storage': (element: Element) => {
			say(i18n[lang]['luggage-storage']);
			navigator.vibrate(100);
		},
		'private-compartment': (element: Element) => {
			say(i18n[lang]['private-compartment']);
			navigator.vibrate(100);
		},
		toilet: (element: Element) => {
			say(i18n[lang].toilet);
			navigator.vibrate(100);
		},
		'telephone-booth': (element: Element) => {
			say(i18n[lang]['telephone-booth']);
			navigator.vibrate(100);
		},
		stairs: (element: Element) => {
			say(i18n[lang].stairs);
			navigator.vibrate(100);
		},
		door: (element: Element) => {
			say(i18n[lang].door);
			navigator.vibrate(100);
		},
        'ski-storage': (element: Element) => {
            say(i18n[lang]['ski-storage']);
            navigator.vibrate(100);
        },
	};

	function identify() {
		const hover = document.elementFromPoint(mousecoords[0], mousecoords[1]);
		const element = hover?.closest('[data-element]');

		if (!element) return;

		if (lastElement === (element || '')) {
			return;
		}
		lastElement = element;

		const type = element.getAttribute('data-element');
		console.log(type);
		if (type && handlers[type]) {
			handlers[type](element);
		}
	}

	let prepped = false;

	function prep() {
		if (landscapewarning) return;
		document.body.requestFullscreen();
		prepped = true;
	}

	function switchwagon(direction: 'next' | 'previous') {
		if (direction === 'next') {
			if (Object.keys(configuration).length === wagon) {
				say('You are already on the last coach');
				return;
			}
			wagon += 1;
		} else {
			if (wagon === 1) {
				say('You are already on the first coach');
				return;
			}
			wagon -= 1;
		}
		say(
			`${i18n[lang].wagon} ${wagon} ${Object.keys(configuration).length === wagon ? '. last coach' : wagon === 1 ? '. first coach' : ''}`
		);
	}

	function switchfloor() {
		floor = floor === 0 ? 1 : 0;
		say(`${floor ? 'second' : 'first'} floor`);
	}
</script>

<div class="main">
	{#if !prepped}
		<p class="sr-only">
			This page is designed as a demo for a virtual tactile map using a train coach map as an
			example. This application requires you to disable your screen reader and use your finger or
			mouse to explore the map. When you hover over an element, the application will read out the
			element's name. When you click on the bottom left or right corners of the screen, you can
			switch between different carriges. Click on bottom center to switch floor (if applicable). Get
			started by clicking (and releasing) on the screen once and then the application will be ready
			for you to explore.
		</p>
		<button class="full" on:click={prep}>Click to start</button>
		{#if landscapewarning}
			<p class="sr-only">Please use this application in landscape mode</p>
		{/if}
	{:else if configuration[wagon] === 'Ed'}
		{#if floor === 0}
			<EdDownstairs />
		{:else}
			<EdUpstairs />
		{/if}
	{:else if configuration[wagon] === 'Edfs'}
		{#if floor === 0}
			<EdfsDownstairs />
		{:else}
			<EdfsDownstairs />
		{/if}
	{/if}

	<div data-button="left" class="bl"></div>
	<div data-button="center" class="bc"></div>
	<div data-button="right" class="br"></div>
</div>

<style>
	* {
		user-select: none;
	}
	.main {
		display: flex;
		justify-content: center;
		align-items: center;
		height: 100dvh;

		font-family: 'Roboto', sans-serif;
		font-size: 1em;
	}

	.sr-only {
		position: absolute;
		clip: rect(1px, 1px, 1px, 1px);
		clip-path: inset(50%);
		height: 1px;
		width: 1px;
		overflow: hidden;
	}

	.full {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		width: 100%;
		height: 100%;
		background: transparent;
		border: none;
		cursor: pointer;

		font-size: 2em;
	}

	.bl {
		position: fixed;
		bottom: 0;
		left: 0;
		width: 120px;
		aspect-ratio: 1;
	}

	.br {
		position: fixed;
		bottom: 0;
		right: 0;
		width: 120px;
		aspect-ratio: 1;
	}

	.bc {
		position: fixed;
		bottom: 0;
		left: 50%;
		transform: translateX(-50%);
		width: 120px;
		aspect-ratio: 1;
	}
</style>
