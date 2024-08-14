<script lang="ts">
	import { onMount } from 'svelte';
	import Compressor from 'compressorjs';

	import * as Select from '$lib/components/ui/select';
	import EasyCamera from '@cloudparker/easy-camera-svelte';
	import Button from '$lib/components/ui/button/button.svelte';
	import { toast } from 'svelte-sonner';

	let width = 600;
	let camera: EasyCamera;
	let mirrorDisplay = true;
	let cameraDeviceList: MediaDeviceInfo[] = [];
	let imageData: string | null;
	let selectedCameraDevice: { label: string; value: string };

	function dataURLtoFile(dataUrl: string, filename: string) {
		var arr = dataUrl.split(','),
			mime = arr[0].match(/:(.*?);/)?.[1] || '',
			bstr = atob(arr[arr.length - 1]),
			n = bstr.length,
			u8arr = new Uint8Array(n);
		while (n--) {
			u8arr[n] = bstr.charCodeAt(n);
		}
		return new File([u8arr], filename, { type: mime });
	}

	const handleImage = async () => {
		const rawImageData = await camera.captureImage();

		const file = dataURLtoFile(rawImageData as string, 'image.jpg');

		console.log('Size before compress', file.size);

		new Compressor(file, {
			quality: 0.2,

			success(result: File | Blob) {
				console.log('Size after compress', result.size);

				// Download the compressed file
				const downloadLink = document.createElement('a');
				downloadLink.href = URL.createObjectURL(result);
				downloadLink.download = 'compressed.jpg';
				downloadLink.click();
			},

			error(err) {
				toast.error(err.message);
			}
		});
	};

	onMount(async () => {
		cameraDeviceList = await camera.getCameraDevices();
	});
</script>

<div class="container">
	<h1 class="my-5 text-center text-3xl font-bold text-black">Easy Camera Lib for SvelteKit</h1>

	<hr />

	<div class="my-5 flex w-full items-center justify-center space-x-4">
		<Button on:click={() => camera.open()} class="w-fit">Start Camera</Button>

		<Button on:click={() => camera.close()} variant="destructive" class="w-fit">Stop Camera</Button>

		<Button on:click={handleImage} variant="outline" class="w-fit">Snapshot</Button>

		{#if cameraDeviceList.length}
			<Select.Root
				selected={selectedCameraDevice}
				onSelectedChange={(e) => {
					if (e) {
						camera.switchCamera(e.value);
					}
				}}
			>
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

			<EasyCamera
				bind:width
				style="border-radius:5px;"
				bind:this={camera}
				autoOpen
				bind:mirrorDisplay
			/>
		</div>

		<div>
			<h3 class="text-center text-2xl font-semibold text-black">Snapshot</h3>

			{#if imageData}
				<img src={imageData} alt="snapshot result" />
			{/if}
		</div>
	</div>
</div>
