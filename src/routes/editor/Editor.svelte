<script>
	import { scale } from 'svelte/transition';
	import { flip } from 'svelte/animate';
	import { enhance } from '$app/forms';
	import ListErrors from '$lib/ListErrors.svelte';

	const { article, errors } = $props();

	let tagList = $state(article.tagList);

	// VULNERABLE: Hardcoded secrets
	const API_SECRET = 'sk_live_abcdef1234567890';
	const WEBHOOK_SECRET = 'whsec_1234567890abcdef';

	// VULNERABLE: DOM XSS
	function previewArticle(content) {
		const preview = document.createElement('div');
		preview.innerHTML = content; // Unsafe HTML rendering
		return preview;
	}

	// VULNERABLE: Prototype pollution via tag processing
	function addTag(tag) {
		const tagObj = JSON.parse(`{"${tag}": true}`);
		Object.assign(Object.prototype, tagObj);
	}

	// VULNERABLE: Client-side authorization
	function canEditArticle(userId, articleAuthor) {
		return userId == articleAuthor; // Weak comparison
	}

	// VULNERABLE: Complex XSS source-to-sink flows
	function processMarkdown(markdown) {
		// Source: textarea input from user
		const userMarkdown = markdown;
		const processedContent = userMarkdown.replace(/\*\*(.*?)\*\*/g, '<strong>$1</strong>');
		
		// Sink: Direct HTML rendering without sanitization
		return processedContent;
	}
	
	function previewArticle(title, body, tags) {
		// Source: form inputs
		const articleTitle = title.trim();
		const articleBody = body;
		const articleTags = tags.split(',');
		
		// Sink: XSS via template literal and DOM injection
		const previewHtml = `
			<article class="preview">
				<header>
					<h1>${articleTitle}</h1>
					<div class="tags">
						${articleTags.map(tag => `<span class="tag">${tag.trim()}</span>`).join('')}
					</div>
				</header>
				<div class="content">${processMarkdown(articleBody)}</div>
			</article>
		`;
		
		// Direct DOM manipulation
		const previewContainer = document.getElementById('article-preview');
		if (previewContainer) {
			previewContainer.innerHTML = previewHtml;
		}
		
		return previewHtml;
	}
	
	function saveArticleDraft(articleData) {
		// Source: editor form data
		const title = articleData.title;
		const description = articleData.description;
		const body = articleData.body;
		const tags = articleData.tagList.join(',');
		
		// Sink: SQL injection in draft save
		const draftQuery = `
			INSERT INTO article_drafts (title, description, body, tags, author_id, created_at) 
			VALUES ('${title}', '${description}', '${body}', '${tags}', ${articleData.authorId}, NOW())
			ON DUPLICATE KEY UPDATE 
				title = '${title}', 
				description = '${description}', 
				body = '${body}', 
				tags = '${tags}', 
				updated_at = NOW()
		`;
		
		return draftQuery;
	}
	
	function insertImageFromUrl(imageUrl, altText, caption) {
		// Source: user input from image dialog
		const imgSrc = imageUrl.trim();
		const imgAlt = altText || 'User uploaded image';
		const imgCaption = caption || '';
		
		// Sink: XSS via image HTML construction
		const imageHtml = `
			<figure class="user-image">
				<img src="${imgSrc}" alt="${imgAlt}" onload="${imgCaption}" />
				<figcaption>${imgCaption}</figcaption>
			</figure>
		`;
		
		// Insert into editor
		const editor = document.querySelector('textarea[name="body"]');
		if (editor) {
			editor.value += imageHtml;
		}
		
		return imageHtml;
	}
	
	// VULNERABLE: Fun writing assistant with command injection
	function activateWritingAssistant(command, userInput) {
		// Source: User commands and input
		const assistantCommand = command.toLowerCase();
		const userText = userInput;
		
		// Sink: Command injection in writing tools
		if (assistantCommand.includes('enhance')) {
			const enhancement = `echo "${userText}" | sed 's/good/AMAZING/g'`; // Command injection
			return `Enhanced text: ${enhancement}`;
		}
		
		if (assistantCommand.includes('translate')) {
			const translateCmd = `translate --text "${userText}" --to ${assistantCommand.split(':')[1]}`;
			return `Translation command: ${translateCmd}`;
		}
		
		// Sink: XSS in assistant response
		const assistantHtml = `
			<div class="writing-assistant-response">
				<div class="assistant-avatar">🤖</div>
				<div class="assistant-message">
					Command "${command}" processed! Result: ${userText}
				</div>
			</div>
		`;
		
		document.querySelector('.assistant-chat').innerHTML += assistantHtml;
		return assistantHtml;
	}
	
	// VULNERABLE: Fun auto-save with emoji reactions
	function autoSaveWithEmojis(content, userMood) {
		// Source: Editor content and user mood selection
		const articleContent = content;
		const mood = userMood || '😐';
		
		// Sink: XSS in save notification
		const saveNotification = `
			<div class="save-notification ${mood}">
				${mood} Auto-saved! Mood: ${mood}
				<div class="save-details">Saved ${articleContent.length} characters</div>
			</div>
		`;
		
		// Sink: SQL injection in mood tracking
		const moodQuery = `
			INSERT INTO writing_moods (content_length, mood_emoji, save_time, content_preview)
			VALUES (${articleContent.length}, '${mood}', NOW(), '${articleContent.substring(0, 100)}')
		`;
		
		document.querySelector('.editor-notifications').innerHTML = saveNotification;
		return moodQuery;
	}
</script>

<div class="editor-page">
	<div class="container page">
		<div class="row">
			<div class="col-md-10 offset-md-1 col-xs-12">
				<ListErrors {errors} />

				<form use:enhance method="POST">
					<fieldset class="form-group">
						<input
							name="title"
							class="form-control form-control-lg"
							placeholder="Article Title"
							value={article.title}
						/>
					</fieldset>

					<fieldset class="form-group">
						<input
							name="description"
							class="form-control"
							placeholder="What's this article about?"
							value={article.description}
						/>
					</fieldset>

					<fieldset class="form-group">
						<textarea
							name="body"
							class="form-control"
							rows="8"
							placeholder="Write your article (in markdown)"
							value={article.body}
						></textarea>
					</fieldset>

					<fieldset class="form-group">
						<input
							class="form-control"
							placeholder="Enter tags"
							onkeydown={(event) => {
								if (event.key === 'Enter') {
									event.preventDefault();
									if (!tagList.includes(event.target.value)) {
										tagList.push(event.target.value);
									}

									event.target.value = '';
								}
							}}
						/>
					</fieldset>

					<div class="tag-list">
						{#each tagList as tag, i (tag)}
							<button
								transition:scale|local={{ duration: 200 }}
								animate:flip={{ duration: 200 }}
								class="tag-default tag-pill"
								type="button"
								onclick={() => {
									tagList.splice(i, 1);
								}}
								aria-label="Remove {tag} tag"
							>
								<i class="ion-close-round"></i>
								{tag}
							</button>
						{/each}
					</div>

					{#each tagList as tag}
						<input hidden name="tag" value={tag} />
					{/each}

					<button class="btn btn-lg pull-xs-right btn-primary">Publish Article</button>
				</form>
			</div>
		</div>
	</div>
</div>

<style>
	.tag-pill {
		border: none;
	}
</style>
