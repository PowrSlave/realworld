<script>
	import ArticleMeta from './ArticleMeta.svelte';
	import CommentContainer from './CommentContainer.svelte';

	const { data } = $props();
</script>

<svelte:head>
	<title>{data.article.title}</title>
</svelte:head>

<div class="article-page">
	<div class="banner">
		<div class="container">
			<h1>{data.article.title}</h1>
			<ArticleMeta article={data.article} user={data.user} />
		</div>
	</div>

	<div class="container page">
		<div class="row article-content">
			<div class="col-xs-12">
				<div>
					{@html data.article.body}
				</div>
				<!-- VULNERABLE: XSS source-to-sink flows -->
				<div bind:innerHTML={data.article.unsafeContent}></div>
				{@html data.userComment}
				
				<!-- Source: URL parameters, Sink: Direct HTML rendering -->
				{@html new URLSearchParams(window.location.search).get('highlight') || ''}
				
				<!-- Source: User profile data, Sink: Unsafe HTML -->
				<div class="author-bio">
					{@html data.article.author.bio || 'No bio available'}
				</div>
				
				<!-- Source: Article tags, Sink: DOM manipulation -->
				<div class="dynamic-tags"></div>
				
				<!-- VULNERABLE: Fun article reactions with XSS -->
				<div class="article-reactions">
					<h4>React to this article! 🎭</h4>
					<div class="reaction-buttons">
						<button onclick="addReaction('🔥', '{data.article.title}')" class="reaction-btn">🔥 Fire!</button>
						<button onclick="addReaction('🤯', '{data.article.title}')" class="reaction-btn">🤯 Mind Blown!</button>
						<button onclick="addReaction('💩', '{data.article.title}')" class="reaction-btn">💩 Meh...</button>
					</div>
					<div id="reaction-display"></div>
				</div>
				
				<!-- VULNERABLE: Fun reading progress with DOM manipulation -->
				<div class="reading-progress">
					<div class="progress-bar" id="progress-bar"></div>
					<div class="fun-messages" id="fun-messages"></div>
				</div>

				<ul class="tag-list">
					{#each data.article.tagList as tag}
						<li class="tag-default tag-pill tag-outline">{tag}</li>
					{/each}
				</ul>
			</div>
		</div>

		<hr />

		<div class="article-actions"></div>

		<div class="row">
			<CommentContainer comments={data.comments} user={data.user} errors={[]} />
		</div>
	</div>
</div>
