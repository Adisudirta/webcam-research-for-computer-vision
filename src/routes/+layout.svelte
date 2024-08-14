<script lang="ts">
	import '../app.pcss';

	import { Toaster } from '$lib/components/ui/sonner';
</script>

<svelte:head>
	<title>Webcam Research by Adi</title>
</svelte:head>

<Toaster position="top-center" />

<div class="container">
	<slot />
</div>
