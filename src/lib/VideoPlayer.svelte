<script>
	import { Icon } from 'svelte-icons-pack';
	import { FaSolidPlay, FaSolidPause } from 'svelte-icons-pack/fa';
	import { VscMute, VscUnmute } from 'svelte-icons-pack/vsc';
	import { RiMediaFullscreenLine, RiMediaFullscreenExitFill } from 'svelte-icons-pack/ri';

	let paused = true;
	let muted = false;
	let currentTime = 0;
	let duration = 0;
	let videoContainer;
	let isFullScreen = false;
	let progressPlayed = 0;

	const togglePlay = () => {
		paused = !paused;
	};

	const toggleAudio = () => {
		muted = !muted;
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

			if (videoContainer.requestFullscreen) {
				videoContainer.requestFullscreen();
			} else if (videoContainer.webkitRequestFullscreen) {
				videoContainer.webkitRequestFullscreen();
			} else if (videoContainer.msRequestFullscreen) {
				videoContainer.msRequestFullscreen();
			}

			isFullScreen = true;
		}
	};

	const handleTimeUpdate = () => {
		const playedPercent = (currentTime / duration) * 100;
		progressPlayed = playedPercent;
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

<div class="bg-gray-200 h-screen flex flex-col items-center justify-center">
	<p class="text-3xl mb-4 underline">Custom Video Player</p>

	<div
		class="relative w-3/4 h-3/4 bg-black flex items-center justify-center overflow-hidden"
		bind:this={videoContainer}
	>
		<video bind:paused bind:muted bind:currentTime bind:duration on:timeupdate={handleTimeUpdate}>
			<track kind="captions" />
			<source src="/demo-video.mp4" type="video/mp4" />
		</video>

		<div class="absolute w-full bottom-0 left-0 bg-[#00000086]">
			<!-- Progress bar -->
			<div class="w-full px-2">
				<div class="h-1 bg-gray-700 rounded-lg relative cursor-pointer">
					<div class="absolute h-full bg-white rounded-lg" style="width: {progressPlayed}%"></div>
				</div>
			</div>

			<!-- Video controls -->
			<div class="h-14 flex justify-between items-center px-4">
				<div class="flex items-center gap-4">
					<button on:click={togglePlay}>
						<Icon src={paused ? FaSolidPlay : FaSolidPause} size="22px" color="white" />
					</button>

					<button on:click={toggleAudio}>
						<Icon src={muted ? VscMute : VscUnmute} size="22px" color="white" />
					</button>

					<div>
						<p class="text-white">{formatVideoTime(currentTime)} / {formatVideoTime(duration)}</p>
					</div>
				</div>

				<div>
					<button on:click={toggleFullScreen}>
						<Icon
							src={isFullScreen ? RiMediaFullscreenExitFill : RiMediaFullscreenLine}
							size="22px"
							color="white"
						/>
					</button>
				</div>
			</div>
		</div>
	</div>
</div>
