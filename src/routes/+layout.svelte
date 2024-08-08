<script>
	import '../app.css';

	import { ModeWatcher } from 'mode-watcher';
	import Header from '$lib/components/header.svelte';
	import Footer from '$lib/components/footer.svelte';
</script>

<ModeWatcher />
<Header></Header>
<slot></slot>
<Footer></Footer>
