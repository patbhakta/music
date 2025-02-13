<script context="module" lang="ts">
	import type { Load } from '@sveltejs/kit';
	export const load: Load = async ({ url, params, fetch }) => {
		const id = url.searchParams.get('id');
		const playlist = url.searchParams.get('list') || undefined;
		// const meta = await get('player', { videoId: id })
		// const data = await meta.body

		if (!id) {
			return { redirect: '/trending', status: 301 };
		}
		const metadata = await fetch(
			`/api/player.json?videoId=${id ? id : ''}${
				playlist ? `&playlistId=${playlist}` : ''
			}`
		);
		const list = await fetch(
			`/api/next.json?videoId=${id ? id : ''}${
				playlist ? `&playlistId=${playlist}` : ''
			}`
		);
		const listData = await list.json();

		const data = await metadata.json();
		const {
			videoDetails: {
				title = '',
				videoId = '',
				thumbnail: { thumbnails = [] } = {}
			} = {}
		} = data;

		return {
			props: {
				title,
				thumbnails,
				videoId,
				playlist,
				related: listData
				// data: listData
			},
			status: 200
		};
	};
</script>

<script>
	export let videoId;
	export let playlist;
	export let thumbnails = [];
	export let title;
	export let related;
	// export let data
	import { goto } from '$app/navigation';
	import Icon from '$lib/components/Icon/Icon.svelte';
	import Listing from '$lib/components/Item/Listing.svelte';
	import list from '$lib/stores/list';
	// $: console.log(related)
	// $: console.log(videoId, playlist, thumbnails, title)
</script>

<svelte:head>
	<meta property="og:title" content={title} />
	<meta property="og:type" content="music.song" />
	<meta property="og:description" content={`Listen to ${title} on Beatbump`} />
	<meta property="og:site_name" content="Beatbump" />
	<meta property="og:image" content={thumbnails[thumbnails.length - 1].url} />

	<meta
		property="og:url"
		content={`https://beatbump.ml/listen?id=${videoId}${
			playlist ? `&list=${playlist}` : ''
		}`}
	/>
	<title>{title} | Beatbump</title>
</svelte:head>
<main>
	<header>
		<div class="image-container">
			<img
				src={thumbnails[thumbnails.length - 1]?.url}
				width={thumbnails[thumbnails.length - 1]?.width}
				height={thumbnails[thumbnails.length - 1]?.height}
				alt={`Thumbnail for ${title}`}
			/>
		</div>
		<div class="body">
			<span class="title h4">{title}</span>
			<button
				on:click={() => {
					list.initAutoMixSession({
						videoId,
						playlistId: playlist ?? related?.currentMixId
					});
				}}
				><Icon name="play" size="1.25em" color="black" /><span class="text"
					>Start Listening</span
				></button
			>
		</div>
	</header>
	<section class="related">
		<span class="h2">Related Tracks</span>
		<div class="results">
			{#each related?.results as result}
				<Listing data={result} />
			{/each}
		</div>
	</section>
	<div class="modal">
		<div class="modal-header" />
		<div class="container" />
	</div>
</main>

<style lang="scss">
	.related {
		display: flex;
		flex-direction: column;
		gap: 0.8rem;
	}

	.body {
		grid-area: body;
		display: flex;
		flex-direction: column;
		align-items: flex-start;
		align-self: center;
		gap: 1.25rem;
	}
	button {
		grid-area: button;
	}
	header {
		display: grid;
		grid-template-areas: 'img body';
		grid-template-columns: 0.5fr 1fr;
		grid-template-rows: 1fr;

		gap: 1rem;
		margin-bottom: 0.8rem;

		background-color: hsl(209deg 20% 27% / 19%);
		border-color: #1b1b1b;
		border-radius: 0.8em;
		border-width: 1px;
		margin: 1em;
		padding: 0.8em;
		transition: all 0.23s cubic-bezier(0.39, 0.575, 0.565, 1);
		@media screen and (min-width: 640px) {
			grid-template-areas:
				'img body'
				'img .';
			grid-template-rows: 1fr 1fr;
		}
	}
	.container {
		place-items: center;
	}

	.image-container {
		max-width: 100%;
		height: auto;
		max-height: 20rem;
		grid-area: img;
	}
	img {
		width: inherit;
		max-width: inherit;
		max-height: inherit;
		width: 100%;
		height: auto;
		object-fit: cover;
	}
</style>
