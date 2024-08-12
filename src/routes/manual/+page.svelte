<script lang="ts">
	import Button from '$lib/components/ui/button/button.svelte';
	import * as Select from '$lib/components/ui/select';

	let isCameraStarted: boolean = false;
	let cameraDeviceList: MediaDeviceInfo[] = [];

	async function getDevices() {
		if (!navigator.mediaDevices || !navigator.mediaDevices.enumerateDevices) {
			console.log('enumerated devices not supported');
			return false;
		}

		await navigator.mediaDevices.getUserMedia({ video: true });

		cameraDeviceList = await navigator.mediaDevices.enumerateDevices();
	}

	async function start() {
		isCameraStarted = true;

		await getDevices();
	}
</script>

<div class="container">
	<h1 class="my-5 text-center text-3xl font-bold text-black">Web API Camera</h1>

	<hr />

	<div class="my-5 flex w-full items-center justify-center space-x-4">
		<Button on:click={start} class="w-fit">Start Camera</Button>

		{#if isCameraStarted && cameraDeviceList.length}
			<Select.Root>
				<Select.Trigger class="w-[300px]">
					<Select.Value placeholder="Select camera devices" />
				</Select.Trigger>

				<Select.Content>
					{#each cameraDeviceList as device}
						<Select.Item value={device.deviceId}>
							{device.label}
						</Select.Item>
					{/each}
				</Select.Content>
			</Select.Root>
		{/if}
	</div>

	<hr />

	<div class="grid grid-cols-2 gap-4">
		<div>
			<h3 class="text-center text-2xl font-semibold text-black">Video Camera</h3>

			{#if isCameraStarted}
				<video id="video" playsinline autoplay>
					<track kind="captions" />
				</video>
			{/if}
		</div>

		<div>
			<h3 class="text-center text-2xl font-semibold text-black">Snapshot</h3>
			<canvas id="canvas"></canvas>
		</div>
	</div>
</div>
