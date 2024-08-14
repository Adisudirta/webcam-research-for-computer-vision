<script lang="ts">
	import { toast } from 'svelte-sonner';
	import { onDestroy, onMount } from 'svelte';

	import * as Select from '$lib/components/ui/select';
	import Button from '$lib/components/ui/button/button.svelte';
	import Compressor from 'compressorjs';

	let isCameraStarted: boolean = false;
	let cameraDeviceList: MediaDeviceInfo[] = [];
	let selectedCameraDevice: {
		label: string;
		value: string;
	};
	let video: HTMLVideoElement;
	let canvas: HTMLCanvasElement;
	let stream: MediaStream | null = null;

	async function getDevices() {
		if (!navigator.mediaDevices || !navigator.mediaDevices.enumerateDevices) {
			toast.error('Enumerated devices not supported');
			return false;
		}

		try {
			await navigator.mediaDevices.getUserMedia({ video: true, audio: false });

			const mediaDevices = await navigator.mediaDevices.enumerateDevices();
			cameraDeviceList = mediaDevices.filter((device) => device.kind === 'videoinput');

			selectedCameraDevice = {
				label: cameraDeviceList[0].label,
				value: cameraDeviceList[0].deviceId
			};
		} catch {
			toast.error('Camera not found');
			return false;
		}
	}

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

	function getSnapshot() {
		if (video && canvas) {
			canvas.width = video.videoWidth;
			canvas.height = video.videoHeight;

			canvas.getContext('2d')?.drawImage(video, 0, 0, canvas.width, canvas.height);

			const fileUrl = canvas.toDataURL('image/jpeg', 0.6);
			const file = dataURLtoFile(fileUrl, 'snapshot.jpg');

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
		}
	}

	function stopCamera() {
		if (stream && video) {
			const tracks = stream.getTracks();
			tracks.forEach((track) => track.stop());
			video.srcObject = null;
			video.removeAttribute('src');
			video.load();
		}
	}

	async function openCamera(deviceId: string) {
		stopCamera();

		try {
			stream = await navigator.mediaDevices.getUserMedia({
				video: {
					deviceId
				}
			});
			video.srcObject = stream;
		} catch (e) {
			console.log('error', e);
			toast.error('Camera not found');
			return false;
		}
	}

	onMount(async () => {
		isCameraStarted = true;
		await getDevices();
	});

	onDestroy(() => {
		stopCamera();
	});
</script>

<div class="container">
	<h1 class="my-5 text-center text-3xl font-bold text-black">Web API Camera</h1>

	<hr />

	<div class="my-5 flex w-full items-center justify-center space-x-4">
		<Button on:click={async () => openCamera(selectedCameraDevice.value)} class="w-fit">
			Start Camera
		</Button>

		<Button disabled={!isCameraStarted} on:click={stopCamera} variant="destructive" class="w-fit">
			Stop Camera
		</Button>

		<Button disabled={!isCameraStarted} on:click={getSnapshot} variant="outline" class="w-fit">
			Snapshot
		</Button>

		{#if cameraDeviceList.length}
			<Select.Root
				selected={selectedCameraDevice}
				onSelectedChange={() => {
					openCamera(selectedCameraDevice.value);
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

			{#if isCameraStarted}
				<video bind:this={video} playsinline autoplay>
					<track kind="captions" />
				</video>
			{/if}
		</div>

		<div>
			<h3 class="text-center text-2xl font-semibold text-black">Snapshot</h3>

			<canvas bind:this={canvas} />
		</div>
	</div>
</div>
