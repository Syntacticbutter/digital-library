<script>
	import '../app.css';

	import { ModeWatcher } from 'mode-watcher';
	import Header from '$lib/components/header.svelte';
	import Footer from '$lib/components/footer.svelte';
	import { Toaster } from '$lib/components/ui/sonner';
</script>

<ModeWatcher />
<Header></Header>
<Toaster />
<slot></slot>
<Footer></Footer>
