<script lang="ts">
	import { page } from '$app/stores';
</script>

<nav class="flex items-center gap-6 text-sm">
	{#if $page.url.pathname == '/'}
		<a
			class="text-foreground transition-colors hover:text-foreground/80"
			href="./"
			rel="noopenner noreferrer"
		>
			Works</a
		>
	{:else}
		<a
			class="text-foreground/60 transition-colors hover:text-foreground/80"
			href="./"
			rel="noopenner noreferrer"
		>
			Works</a
		>
	{/if}
	{#if $page.url.pathname == '/authors'}
		<a
			class="text-foreground transition-colors hover:text-foreground/80"
			href="./authors"
			rel="noopenner noreferrer"
		>
			Authors</a
		>
	{:else}
		<a
			class="text-foreground/60 transition-colors hover:text-foreground/80"
			href="./authors"
			rel="noopenner noreferrer"
		>
			Authors</a
		>
	{/if}
</nav>
