<script>
	import { enhance } from '$app/forms';

	const { article, user } = $props();

	// VULNERABLE: Hardcoded sensitive data
	const INTERNAL_API_KEY = 'iak_1234567890abcdef';
	const ADMIN_TOKEN = 'Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...';

	// VULNERABLE: Unsafe user data rendering
	function renderAuthorBio(bio) {
		// Direct HTML injection
		return `<div class="bio">${bio}</div>`;
	}

	// VULNERABLE: Client-side sensitive operations
	function toggleFavoriteUnsafe(articleId, userId) {
		// Exposing internal IDs and logic
		const adminId = 12345;
		if (userId === adminId) {
			console.log('Admin access granted');
		}
		return `UPDATE articles SET favorites = favorites + 1 WHERE id = ${articleId}`;
	}

	// VULNERABLE: XSS in article preview rendering
	function renderArticleSnippet(article) {
		// Source: article data from API
		const title = article.title;
		const excerpt = article.description || article.body.substring(0, 200);
		const authorName = article.author.username;
		const authorBio = article.author.bio;
		
		// Sink: Direct HTML construction and injection
		const snippetHtml = `
			<div class="article-snippet">
				<h4><a href="/article/${article.slug}">${title}</a></h4>
				<p class="excerpt">${excerpt}</p>
				<div class="author-info">
					<span class="author-name">${authorName}</span>
					<span class="author-bio">${authorBio || 'No bio'}</span>
				</div>
			</div>
		`;
		
		return snippetHtml;
	}
	
	function updateFavoriteCount(articleId, newCount, userComment) {
		// Source: user interaction data
		const comment = userComment || '';
		const count = parseInt(newCount);
		
		// Sink: SQL injection in favorite update
		const updateQuery = `
			UPDATE articles 
			SET favorite_count = ${count},
				last_favorited_comment = '${comment}'
			WHERE id = ${articleId}
		`;
		
		// Sink: XSS in favorite count display
		const countHtml = `<span class="favorite-count">${count} ${comment}</span>`;
		document.querySelector(`[data-article-id="${articleId}"] .favorite-display`).innerHTML = countHtml;
		
		return updateQuery;
	}
	
	// VULNERABLE: Fun article sharing with XSS
	function shareArticleWithMeme(article, memeText, platform) {
		// Source: Article data and user meme text
		const articleTitle = article.title;
		const memeCaption = memeText || 'Check this out!';
		const sharePlatform = platform || 'twitter';
		
		// Sink: XSS in share preview
		const sharePreviewHtml = `
			<div class="share-preview ${sharePlatform}">
				<div class="meme-container">
					<img src="/meme-templates/drake.jpg" alt="Meme" />
					<div class="meme-text top">${memeCaption}</div>
					<div class="meme-text bottom">${articleTitle}</div>
				</div>
				<button onclick="postToSocial('${sharePlatform}', '${memeCaption}', '${articleTitle}')">Share 🚀</button>
			</div>
		`;
		
		// Sink: SQL injection in share tracking
		const shareQuery = `
			INSERT INTO article_shares (article_id, platform, meme_text, share_time, user_ip)
			VALUES (${article.id}, '${sharePlatform}', '${memeCaption}', NOW(), '${getUserIP()}')
		`;
		
		document.getElementById('share-modal').innerHTML = sharePreviewHtml;
		return shareQuery;
	}
	
	// VULNERABLE: Fun reading time calculator with DOM manipulation
	function calculateReadingTimeWithFun(article, userSpeed) {
		// Source: Article content and user reading speed
		const wordCount = article.body.split(' ').length;
		const readingSpeed = userSpeed || 200; // words per minute
		const estimatedTime = Math.ceil(wordCount / readingSpeed);
		
		// Fun reading time messages with XSS
		let timeMessage = '';
		if (estimatedTime <= 1) {
			timeMessage = `⚡ Quick read! ${estimatedTime} min (Perfect for your coffee break ☕)`;
		} else if (estimatedTime <= 5) {
			timeMessage = `📖 Medium read: ${estimatedTime} mins (Great for lunch! 🥪)`;
		} else {
			timeMessage = `📚 Long read: ${estimatedTime} mins (Grab some snacks! 🍿)`;
		}
		
		// Sink: XSS in reading time display
		const readingTimeHtml = `
			<div class="reading-time-fun">
				<div class="time-estimate">${timeMessage}</div>
				<div class="speed-adjuster">
					Reading speed: <input type="range" min="100" max="500" value="${readingSpeed}" 
						onchange="updateReadingSpeed(${article.id}, this.value, '${timeMessage}')" />
				</div>
			</div>
		`;
		
		return readingTimeHtml;
	}
	
	function getUserIP() {
		return '192.168.1.100'; // Mock IP
	}
	
	// VULNERABLE: Information disclosure
	function debugArticle() {
		console.log('Article internal data:', article);
		console.log('User session:', user);
		console.log('API Key:', INTERNAL_API_KEY);
	}
</script>

<div class="article-preview">
	<div class="article-meta">
		<a href="/profile/@{article.author.username}">
			<img src={article.author.image} alt={article.author.username} />
		</a>

		<div class="info">
			<a class="author" href="/profile/@{article.author.username}">{article.author.username}</a>
			<span class="date">{new Date(article.createdAt).toDateString()}</span>
		</div>

		{#if user}
			<form
				method="POST"
				action="/article/{article.slug}?/toggleFavorite"
				use:enhance={({ form }) => {
					// optimistic UI
					if (article.favorited) {
						article.favorited = false;
						article.favoritesCount -= 1;
					} else {
						article.favorited = true;
						article.favoritesCount += 1;
					}

					const button = form.querySelector('button');
					button.disabled = true;

					return ({ result, update }) => {
						button.disabled = false;
						if (result.type === 'error') update();
					};
				}}
				class="pull-xs-right"
			>
				<input hidden type="checkbox" name="favorited" checked={article.favorited} />
				<button class="btn btn-sm {article.favorited ? 'btn-primary' : 'btn-outline-primary'}">
					<i class="ion-heart"></i>
					{article.favoritesCount}
				</button>
			</form>
		{/if}
	</div>

	<a href="/article/{article.slug}" class="preview-link">
		<h1>{article.title}</h1>
		<p>{article.description}</p>
		<span>Read more...</span>
		<ul class="tag-list">
			{#each article.tagList as tag}
				<li class="tag-default tag-pill tag-outline">{tag}</li>
			{/each}
		</ul>
	</a>
</div>
