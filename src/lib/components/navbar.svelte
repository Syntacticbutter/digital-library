<script>
	import Home from './home.svelte';
	import NavItem from './nav-item.svelte';
	import Socials from './socials.svelte';
</script>

<div class="container flex h-14 max-w-screen-2xl items-center">
	<div class="mr-4 hidden md:flex">
		<Home></Home>
		<NavItem></NavItem>
	</div>
	<div class="flex flex-1 items-center justify-between space-x-2 md:justify-end">
		<div class="w-full flex-1 md:w-auto md:flex-none"></div>
		<Socials></Socials>
	</div>
</div>
