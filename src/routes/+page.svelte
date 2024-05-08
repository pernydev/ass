<script lang="ts">
	import Downstairs from '$lib/components/intercity/Ed/Downstairs.svelte';
	import { onMount } from 'svelte';

	let mousecoords = [0, 0];
	let lastElement: Element;
	let lastTouchTime = 0;

	function handleMouseMove(event: MouseEvent) {
		mousecoords = [event.clientX, event.clientY];
		identify();
	}

	function handleFingerMove(event: TouchEvent) {
		mousecoords = [event.touches[0].clientX, event.touches[0].clientY];
		identify();
	}

	function keyboardListener(event: KeyboardEvent) {
		if (event.key === 'i') {
			identify();
		}
	}

	onMount(() => {
		window.addEventListener('mousemove', handleMouseMove);
		window.addEventListener('keydown', keyboardListener);
		window.addEventListener('touchmove', handleFingerMove);

		return () => {
			window.removeEventListener('mousemove', handleMouseMove);
			window.removeEventListener('keydown', keyboardListener);
			window.removeEventListener('touchmove', handleFingerMove);
		};
	});

    const handlers: Record<string, (element: Element) => void> = {
        seat: (element: Element) => {
            let seat = element.id.split('seat_')[1];
            if (seat.endsWith("_shape")) {
                seat = seat.split('_shape')[0];
            }

            const utterance = new SpeechSynthesisUtterance(`Seat ${seat}`);
            window.speechSynthesis.speak(utterance);
            navigator.vibrate(50);
        },
        table: (element: Element) => {
            const utterance = new SpeechSynthesisUtterance(`Table`);
            window.speechSynthesis.speak(utterance);
            navigator.vibrate(50);
        },
        "luggage-storage": (element: Element) => {
            const utterance = new SpeechSynthesisUtterance(`Luggage storage`);
            window.speechSynthesis.speak(utterance);
            navigator.vibrate(50);
        },
        "private-compartment": (element: Element) => {
            const utterance = new SpeechSynthesisUtterance(`Private compartment`);
            window.speechSynthesis.speak(utterance);
            navigator.vibrate(50);
        },
        "toilet": (element: Element) => {
            const utterance = new SpeechSynthesisUtterance(`Toilet`);
            window.speechSynthesis.speak(utterance);
            navigator.vibrate(50);
        },
        "telephone-booth": (element: Element) => {
            const utterance = new SpeechSynthesisUtterance(`Telephone booth`);
            window.speechSynthesis.speak(utterance);
            navigator.vibrate(50);
        },
        "stairs": (element: Element) => {
            const utterance = new SpeechSynthesisUtterance(`Stairs`);
            window.speechSynthesis.speak(utterance);
            navigator.vibrate(50);
        },
        "door": (element: Element) => {
            const utterance = new SpeechSynthesisUtterance(`Door`);
            window.speechSynthesis.speak(utterance);
            navigator.vibrate(50);
        },
    }

	function identify() {
		const hover = document.elementFromPoint(mousecoords[0], mousecoords[1]);
		const element = hover?.closest('[data-element]');

		if (!element) return;

		if (lastElement === (element || '')) {
			return;
		}
		lastElement = element;

		window.speechSynthesis.cancel();    

        const type = element.getAttribute('data-element');
        console.log(type);
        if (type && handlers[type]) {
            handlers[type](element);
        }
	}

	let prepped = false;

	function prep() {
		prepped = true;

		console.log(window.speechSynthesis.getVoices());

		const utterance = new SpeechSynthesisUtterance('Prepped');
		window.speechSynthesis.speak(utterance);
	}
</script>

{#if !prepped}
	<button on:click={prep}> Prep </button>
{:else}
	<Downstairs />
{/if}
