<script>
	import { Icon } from 'svelte-icons-pack';
	import { FaSolidPlay, FaSolidPause } from 'svelte-icons-pack/fa';
	import { VscMute, VscUnmute } from 'svelte-icons-pack/vsc';
	import { RiMediaFullscreenLine, RiMediaFullscreenExitFill } from 'svelte-icons-pack/ri';
	import { onMount } from 'svelte';

	let paused;
	let muted;
	let isFullScreen = false;
	let videoContainerElement;
	let currentTime = 0;
	let duration = 0;
	let progressbarElement;
	let progressPlayed = 0;
	let playerContainer =
		typeof window !== 'undefined' ? document.querySelector('.player-container') : null;
	let showControls = true;
	let controlsTimeout;

	const togglePlay = () => {
		paused = !paused;
	};

	const toggleAudio = () => {
		muted = !muted;
	};

	const toggleFullScreen = () => {
		if (
			document.fullscreenElement ||
			document.webkitFullscreenElement ||
			document.msFullscreenElement
		) {
			// Exit fullscreen mode
			if (document.exitFullscreen) {
				document.exitFullscreen();
			} else if (document.webkitExitFullscreen) {
				document.webkitExitFullscreen();
			} else if (document.msExitFullscreen) {
				document.msExitFullscreen();
			}
			isFullScreen = false;
		} else {
			// Enter fullscreen mode
			if (videoContainerElement.requestFullscreen) {
				videoContainerElement.requestFullscreen();
			} else if (videoContainerElement.webkitRequestFullscreen) {
				videoContainerElement.webkitRequestFullscreen();
			} else if (videoContainerElement.msRequestFullscreen) {
				videoContainerElement.msRequestFullscreen();
			}
			isFullScreen = true;
		}
	};

	const formatVideoTime = (timeInSeconds) => {
		const hour = Math.floor(timeInSeconds / 3600)
			.toString()
			.padStart(2, '0');
		const minutes = Math.floor((timeInSeconds % 3600) / 60)
			.toString()
			.padStart(2, '0');
		const seconds = Math.floor(timeInSeconds % 60)
			.toString()
			.padStart(2, '0');

		return `${hour > 0 ? hour + ':' : ''}${minutes}:${seconds}`;
	};

	const handleTimeUpdate = () => {
		const playedPercent = (currentTime / duration) * 100;
		progressPlayed = playedPercent;
	};

	const scrubVideo = (event) => {
		const rect = progressbarElement.getBoundingClientRect();
		let relX = event.pageX - (rect.left + document.body.scrollLeft);

		if (relX < 0) {
			relX = 0;
		} else if (relX > progressbarElement.offsetWidth) {
			relX = progressbarElement.offsetWidth;
		}

		const relXPercent = (relX / progressbarElement.offsetWidth) * 100;
		const timeInSeconds = (duration * relXPercent) / 100;
		const playedPercent = (timeInSeconds / duration) * 100;

		progressPlayed = playedPercent;
		currentTime = timeInSeconds;
	};

	const onProgressDragStop = () => {
		paused = false;
		playerContainer.removeEventListener('mousemove', scrubVideo);
		document.removeEventListener('mouseup', onProgressDragStop);
	};

	const handleProgressDrag = (event) => {
		paused = true;
		scrubVideo(event);
		playerContainer.addEventListener('mousemove', scrubVideo);
		document.addEventListener('mouseup', onProgressDragStop);
	};

	const onKeyDown = (event) => {
		if (event.key === 'f') {
			toggleFullScreen();
		}

		if (event.key === ' ') {
			togglePlay();
		}
	};
</script>

<svelte:window on:keydown={onKeyDown} />

<div class="bg-gray-200 h-screen flex items-center justify-center">
	<div
		class="relative w-3/4 h-3/4 bg-black flex items-center justify-center overflow-hidden player-container"
		bind:this={videoContainerElement}
	>
		<video
			class="h-auto w-auto max-w-full"
			bind:paused
			bind:muted
			bind:duration
			bind:currentTime
			on:timeupdate={handleTimeUpdate}
			on:click={togglePlay}
		>
			<track kind="captions" />
			<source src="/demo-video.mp4" type="video/mp4" />
		</video>

		{#if showControls}
			<div class="absolute w-full bottom-0 left-0 bg-[#00000086]">
				<div class="w-full px-2">
					<div
						class="h-1 bg-gray-700 rounded-lg relative cursor-pointer"
						role="presentation"
						bind:this={progressbarElement}
						on:mousedown={handleProgressDrag}
					>
						<div
							class="absolute h-full bg-white rounded-lg"
							role="progressbar"
							aria-label="played"
							style="width: {progressPlayed}%"
						></div>
					</div>
				</div>

				<div class="h-14 flex justify-between items-center px-4">
					<div class="flex items-center gap-4">
						<button on:click={togglePlay}>
							<Icon color="#ffffff" src={paused ? FaSolidPlay : FaSolidPause} size="22px" />
						</button>

						<button on:click={toggleAudio}>
							<Icon src={muted ? VscMute : VscUnmute} color="#ffffff" size="22px" />
						</button>

						<div>
							<p class="text-white">{formatVideoTime(currentTime)} / {formatVideoTime(duration)}</p>
						</div>
					</div>

					<div>
						<button on:click={toggleFullScreen}>
							<Icon
								color="#ffffff"
								src={isFullScreen ? RiMediaFullscreenExitFill : RiMediaFullscreenLine}
								size="22px"
							/>
						</button>
					</div>
				</div>
			</div>
		{/if}
	</div>
</div>
