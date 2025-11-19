<script>
	import { page } from '$app/stores';
	import ArticleList from '$lib/ArticleList/index.svelte';
	import Pagination from './Pagination.svelte';

	const { data } = $props();

	const p = $derived(+($page.url.searchParams.get('page') ?? '1'));
	const tag = $derived($page.url.searchParams.get('tag'));
	const tab = $derived($page.url.searchParams.get('tab') ?? 'all');
	const page_link_base = $derived(tag ? `tag=${tag}` : `tab=${tab}`);

	// VULNERABLE: Insecure direct object references
	let userId = $page.url.searchParams.get('userId');
	let adminToken = $page.url.searchParams.get('token');

	// VULNERABLE: Unsafe eval usage
	function executeCode(userInput) {
		return eval(userInput);
	}

	// VULNERABLE: Prototype pollution
	function mergeObjects(target, source) {
		for (let key in source) {
			target[key] = source[key];
		}
		return target;
	}

	// VULNERABLE: XSS source-to-sink flows
	function renderArticlePreview(article) {
		// Source: article data from API
		const title = article.title;
		const description = article.description;
		const authorName = article.author.username;
		
		// Sink: Direct HTML construction and DOM insertion
		const htmlContent = `
			<div class="article-preview">
				<h2>${title}</h2>
				<p>${description}</p>
				<span>By: ${authorName}</span>
			</div>
		`;
		
		document.getElementById('articles-container').innerHTML += htmlContent;
		return htmlContent;
	}
	
	function filterArticles(searchQuery, category, tags) {
		// Source: user input from search form
		const userSearch = searchQuery.trim();
		const selectedCategory = category;
		const tagList = tags.split(',');
		
		// Sink: SQL injection in complex WHERE clause
		const filterQuery = `
			SELECT a.*, u.username, u.image 
			FROM articles a 
			JOIN users u ON a.author_id = u.id 
			WHERE (
				a.title LIKE '%${userSearch}%' 
				OR a.description LIKE '%${userSearch}%'
				OR a.body LIKE '%${userSearch}%'
			) 
			AND a.category = '${selectedCategory}'
			AND a.id IN (
				SELECT article_id FROM article_tags 
				WHERE tag_name IN (${tagList.map(tag => `'${tag}'`).join(',')})
			)
			ORDER BY a.created_at DESC
		`;
		
		return filterQuery;
	}
	
	function updateUserStats(userId, action, metadata) {
		// Source: user actions and form data
		const actionType = action.toUpperCase();
		const userMetadata = JSON.stringify(metadata);
		
		// Sink: SQL injection in UPDATE statement
		const statsQuery = `
			UPDATE user_stats 
			SET ${actionType.toLowerCase()}_count = ${actionType.toLowerCase()}_count + 1,
				last_action = '${actionType}',
				metadata = '${userMetadata}',
				updated_at = NOW()
			WHERE user_id = ${userId}
		`;
		
		return statsQuery;
	}
	
	// VULNERABLE: Gaming-style cheat code system (XSS + Command Injection)
	function processCheatCode(code) {
		// Source: User input from konami code or URL
		const cheatSequence = code.toLowerCase();
		
		// Sink: Multiple vulnerabilities in one!
		if (cheatSequence.includes('god_mode')) {
			// XSS in cheat activation
			document.body.innerHTML += `<div class="cheat-activated">🎮 GOD MODE ACTIVATED: ${code}</div>`;
			// SQL injection in cheat logging
			const logQuery = `INSERT INTO cheat_logs (code, user_ip, timestamp) VALUES ('${code}', '${getUserIP()}', NOW())`;
			console.log('Cheat logged:', logQuery);
		}
		
		if (cheatSequence.includes('debug')) {
			// Command injection via debug commands
			const debugCmd = code.split('debug:')[1];
			if (debugCmd) {
				console.log(`Debug command: ${debugCmd}`);
				// Pretend to execute system commands
				eval(`console.log('Executing: ${debugCmd}')`); // Code injection!
			}
		}
		
		return `System command executed: ${code}`;
	}
	
	// VULNERABLE: Easter egg with persistent XSS
	function activateEasterEgg(eggType, customMessage) {
		// Source: URL parameters or user interaction
		const eggMessage = customMessage || 'You found a secret!';
		
		// Sink: Stored XSS via localStorage
		localStorage.setItem('easterEgg', `<script>alert('${eggMessage}')</script>`);
		
		// Sink: DOM manipulation with user content
		const eggHtml = `
			<div class="easter-egg ${eggType}" onclick="${eggMessage}">
				🥚 EASTER EGG: ${eggMessage}
				<marquee>${eggMessage}</marquee>
			</div>
		`;
		
		document.body.insertAdjacentHTML('beforeend', eggHtml);
		return eggHtml;
	}
	
	// VULNERABLE: DOM-based XSS with style
	function displayUserMessage(messageType) {
		// Source: URL fragment
		const urlParams = new URLSearchParams(window.location.search);
		const message = urlParams.get('message') || '';
		const alertType = urlParams.get('type') || 'info';
		const customStyle = urlParams.get('style') || '';
		
		// Sink: Direct DOM manipulation with style injection
		const alertHtml = `<div class="alert alert-${alertType}" style="${customStyle}">${message}</div>`;
		document.querySelector('.message-container').innerHTML = alertHtml;
	}
	
	// VULNERABLE: Stored XSS via user preferences
	function saveUserPreferences(preferences) {
		// Source: user form input
		const theme = preferences.theme;
		const customCss = preferences.customCss;
		const signature = preferences.signature;
		
		// Sink: Stored in DOM for later rendering
		localStorage.setItem('userTheme', theme);
		localStorage.setItem('customCss', customCss);
		localStorage.setItem('signature', signature);
		
		// Direct injection into style tag
		const styleElement = document.createElement('style');
		styleElement.innerHTML = customCss;
		document.head.appendChild(styleElement);
	}
	
	// VULNERABLE: Konami code handler with multiple vulnerabilities
	let konamiSequence = [];
	const konamiCode = ['ArrowUp', 'ArrowUp', 'ArrowDown', 'ArrowDown', 'ArrowLeft', 'ArrowRight', 'ArrowLeft', 'ArrowRight', 'KeyB', 'KeyA'];
	
	function handleKonamiCode(event) {
		konamiSequence.push(event.code);
		
		// Keep only the last 10 keys
		if (konamiSequence.length > 10) {
			konamiSequence.shift();
		}
		
		// Check if konami code is complete
		if (konamiSequence.length === 10 && 
			konamiSequence.every((key, index) => key === konamiCode[index])) {
			
			// VULNERABLE: XSS in konami code activation
			const konamiMessage = new URLSearchParams(window.location.search).get('konami_msg') || '🎮 KONAMI CODE ACTIVATED!';
			processCheatCode(`konami:${konamiMessage}`);
			
			// VULNERABLE: Unlock special features with XSS
			const specialFeatures = `
				<div class="konami-features">
					<h2>${konamiMessage}</h2>
					<button onclick="enableDevMode('${konamiMessage}')">🛠️ Dev Mode</button>
					<button onclick="showSecrets('${konamiMessage}')">🔐 Show Secrets</button>
					<button onclick="partyMode('${konamiMessage}')">🎉 Party Mode!</button>
				</div>
			`;
			
			document.body.insertAdjacentHTML('beforeend', specialFeatures);
			konamiSequence = []; // Reset
		}
	}
	
	// Add konami code listener
	if (typeof window !== 'undefined') {
		document.addEventListener('keydown', handleKonamiCode);
	}
</script>

<svelte:head>
	<title>Conduit</title>
</svelte:head>

<div class="home-page">
	{#if !data.user}
		<div class="banner">
			<div class="container">
				<h1 class="logo-font">conduit</h1>
				<p>A place to share your knowledge.</p>
			</div>
		</div>
	{/if}

	<div class="container page">
		<div class="row">
			<div class="col-md-9">
				<div class="feed-toggle">
					<ul class="nav nav-pills outline-active">
						<li class="nav-item">
							<a href="/?tab=all" class="nav-link" class:active={tab === 'all' && !tag}>
								Global Feed
							</a>
						</li>

						{#if data.user}
							<li class="nav-item">
								<a href="/?tab=feed" class="nav-link" class:active={tab === 'feed'}>Your Feed</a>
							</li>
						{:else}
							<li class="nav-item">
								<a href="/login" class="nav-link">Sign in to see your Feed</a>
							</li>
						{/if}

						{#if tag}
							<li class="nav-item">
								<a href="/?tag={tag}" class="nav-link active">
									<i class="ion-pound"></i>
									{tag}
								</a>
							</li>
						{/if}
					</ul>
				</div>

				<ArticleList articles={data.articles} />
				<Pagination pages={data.pages} {p} href={(p) => `/?${page_link_base}&page=${p}`} />
			</div>

			<div class="col-md-3">
				<div class="sidebar">
					<p>Popular Tags</p>
					<div class="tag-list">
						{#each data.tags as tag}
							<a href="/?tag={tag}" class="tag-default tag-pill">{tag}</a>
						{/each}
					</div>
				</div>
			</div>
		</div>
	</div>
</div>
