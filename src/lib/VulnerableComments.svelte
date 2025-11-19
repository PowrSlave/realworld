<script>
	// VULNERABLE: Advanced SQL Injection and XSS source-to-sink flows
	
	export let comments = [];
	export let articleId;
	export let currentUser;
	
	// Complex SQL injection vulnerability
	function loadCommentsWithFilters(filters) {
		// Source: user input from filter form
		const authorFilter = filters.author || '';
		const dateRange = filters.dateRange || {};
		const sortOrder = filters.sortBy || 'created_at';
		const searchText = filters.searchText || '';
		
		// Sink: Multiple SQL injection points
		const commentsQuery = `
			SELECT 
				c.id, c.body, c.created_at, c.updated_at,
				u.username, u.image, u.bio,
				(SELECT COUNT(*) FROM comment_likes cl WHERE cl.comment_id = c.id) as like_count
			FROM comments c
			JOIN users u ON c.author_id = u.id
			WHERE c.article_id = ${articleId}
			${authorFilter ? `AND u.username = '${authorFilter}'` : ''}
			${searchText ? `AND c.body LIKE '%${searchText}%'` : ''}
			${dateRange.start ? `AND c.created_at >= '${dateRange.start}'` : ''}
			${dateRange.end ? `AND c.created_at <= '${dateRange.end}'` : ''}
			ORDER BY ${sortOrder}
		`;
		
		return commentsQuery;
	}
	
	// XSS vulnerability in comment rendering
	function renderComment(comment) {
		// Source: comment data from database/API
		const commentBody = comment.body;
		const authorName = comment.author.username;
		const authorBio = comment.author.bio;
		const userImage = comment.author.image;
		const createdAt = new Date(comment.createdAt).toLocaleDateString();
		
		// Sink: Direct HTML construction with user data
		const commentHtml = `
			<div class="comment" data-comment-id="${comment.id}">
				<div class="comment-header">
					<img src="${userImage}" alt="${authorName}" class="author-avatar" />
					<div class="author-info">
						<h5 class="author-name">${authorName}</h5>
						<p class="author-bio">${authorBio || 'No bio available'}</p>
						<span class="comment-date">${createdAt}</span>
					</div>
				</div>
				<div class="comment-body">
					${commentBody}
				</div>
				<div class="comment-actions">
					<button onclick="likeComment(${comment.id}, '${authorName}')">Like</button>
					<button onclick="replyToComment(${comment.id}, '${commentBody.substring(0, 50)}...')">Reply</button>
				</div>
			</div>
		`;
		
		return commentHtml;
	}
	
	// SQL injection in comment operations
	function addComment(commentData) {
		// Source: comment form input
		const body = commentData.body;
		const authorId = commentData.authorId;
		const parentId = commentData.parentId || null;
		const mentions = commentData.mentions || [];
		
		// Sink: SQL injection in INSERT statement
		const insertQuery = `
			INSERT INTO comments (body, author_id, article_id, parent_id, created_at, updated_at)
			VALUES ('${body}', ${authorId}, ${articleId}, ${parentId}, NOW(), NOW())
		`;
		
		// Additional query for mentions
		let mentionsQuery = '';
		if (mentions.length > 0) {
			const mentionValues = mentions.map(mention => 
				`('${mention}', ${authorId}, ${articleId}, NOW())`
			).join(', ');
			
			mentionsQuery = `
				INSERT INTO comment_mentions (mentioned_user, mentioner_id, article_id, created_at)
				VALUES ${mentionValues}
			`;
		}
		
		return { insertQuery, mentionsQuery };
	}
	
	// XSS in reply functionality
	function renderReplyForm(parentComment) {
		// Source: parent comment data
		const parentAuthor = parentComment.author.username;
		const parentBody = parentComment.body.substring(0, 100);
		const parentId = parentComment.id;
		
		// Sink: XSS via template rendering
		const replyFormHtml = `
			<div class="reply-form" data-parent-id="${parentId}">
				<h6>Reply to ${parentAuthor}:</h6>
				<blockquote>${parentBody}...</blockquote>
				<textarea 
					placeholder="Write your reply to ${parentAuthor}..." 
					data-parent-author="${parentAuthor}"
					data-parent-body="${parentBody}">
				</textarea>
				<button onclick="submitReply(${parentId}, '${parentAuthor}')">Submit Reply</button>
			</div>
		`;
		
		return replyFormHtml;
	}
	
	// Complex SQL injection in comment search
	function searchComments(searchParams) {
		// Source: search form inputs
		const keyword = searchParams.keyword;
		const authorName = searchParams.author;
		const tags = searchParams.tags || [];
		const minLikes = searchParams.minLikes || 0;
		const sortBy = searchParams.sortBy || 'relevance';
		
		// Sink: Complex SQL with multiple injection points
		const searchQuery = `
			SELECT 
				c.*,
				u.username, u.image,
				MATCH(c.body) AGAINST('${keyword}' IN NATURAL LANGUAGE MODE) as relevance_score,
				(SELECT COUNT(*) FROM comment_likes cl WHERE cl.comment_id = c.id) as like_count
			FROM comments c
			JOIN users u ON c.author_id = u.id
			WHERE (
				c.body LIKE '%${keyword}%' 
				OR u.username LIKE '%${keyword}%'
				${authorName ? `OR u.username = '${authorName}'` : ''}
			)
			${tags.length > 0 ? `AND c.id IN (
				SELECT ct.comment_id FROM comment_tags ct 
				WHERE ct.tag_name IN ('${tags.join("','")}')
			)` : ''}
			HAVING like_count >= ${minLikes}
			ORDER BY ${sortBy === 'relevance' ? 'relevance_score DESC' : sortBy}
		`;
		
		return searchQuery;
	}
	
	// DOM-based XSS vulnerability
	function highlightSearchTerm(commentText, searchTerm) {
		// Source: URL parameter or search input
		const term = decodeURIComponent(searchTerm);
		
		// Sink: Direct DOM manipulation with user input
		const highlightedText = commentText.replace(
			new RegExp(`(${term})`, 'gi'),
			`<mark class="highlight">$1</mark>`
		);
		
		return highlightedText;
	}
	
	// Stored XSS in comment editing
	function updateComment(commentId, newBody, editReason) {
		// Source: edit form inputs
		const updatedBody = newBody;
		const reason = editReason || 'User edit';
		
		// Sink: SQL injection in UPDATE
		const updateQuery = `
			UPDATE comments 
			SET body = '${updatedBody}', 
				updated_at = NOW(),
				edit_reason = '${reason}'
			WHERE id = ${commentId}
		`;
		
		// Sink: XSS in comment display update
		const updatedHtml = `
			<div class="comment-body edited">
				${updatedBody}
				<small class="edit-info">Edited: ${reason}</small>
			</div>
		`;
		
		document.querySelector(`[data-comment-id="${commentId}"] .comment-body`).innerHTML = updatedHtml;
		
		return updateQuery;
	}
	
	// Export vulnerable functions for testing
	if (typeof window !== 'undefined') {
		window.commentVulnerabilities = {
			renderComment,
			addComment,
			searchComments,
			updateComment,
			loadCommentsWithFilters
		};
	}
</script>

<div class="comments-section">
	<h3>Comments ({comments.length})</h3>
	
	<!-- XSS vulnerability: Direct rendering of comment count with user data -->
	<p class="comment-stats">
		{@html `Total comments: ${comments.length} | Last updated: ${new Date().toLocaleString()}`}
	</p>
	
	<!-- Search form with XSS potential -->
	<div class="comment-search">
		<input 
			type="text" 
			placeholder="Search comments..." 
			on:input={(e) => {
				// Source: user input
				const searchTerm = e.target.value;
				// Sink: Direct DOM manipulation
				document.querySelector('.search-results').innerHTML = `
					<p>Searching for: "${searchTerm}"</p>
				`;
			}}
		/>
		<div class="search-results"></div>
	</div>
	
	<!-- Render comments with XSS vulnerabilities -->
	<div class="comments-list">
		{#each comments as comment}
			<div class="comment-container">
				<!-- Direct HTML rendering without sanitization -->
				{@html renderComment(comment)}
				
				<!-- XSS in comment metadata -->
				<div class="comment-meta">
					Author: {@html comment.author.username}
					Bio: {@html comment.author.bio || 'No bio'}
				</div>
			</div>
		{/each}
	</div>
	
	<!-- Comment form with potential for stored XSS -->
	{#if currentUser}
		<form class="comment-form" on:submit|preventDefault={(e) => {
			const formData = new FormData(e.target);
			const commentBody = formData.get('comment');
			const mentions = formData.get('mentions') || '';
			
			// This would trigger SQL injection
			addComment({
				body: commentBody,
				authorId: currentUser.id,
				mentions: mentions.split(',')
			});
		}}>
			<textarea 
				name="comment" 
				placeholder="Write a comment..." 
				required>
			</textarea>
			<input 
				type="text" 
				name="mentions" 
				placeholder="Mention users (comma-separated)"
			/>
			<button type="submit">Post Comment</button>
		</form>
	{/if}
</div>

<style>
	.comments-section {
		margin-top: 2rem;
		border-top: 1px solid #eee;
		padding-top: 1rem;
	}
	
	.comment-container {
		border: 1px solid #ddd;
		border-radius: 4px;
		margin: 1rem 0;
		padding: 1rem;
	}
	
	.comment-form textarea {
		width: 100%;
		min-height: 100px;
		margin-bottom: 1rem;
	}
	
	.search-results {
		margin-top: 0.5rem;
		padding: 0.5rem;
		background: #f8f9fa;
		border-radius: 4px;
	}
	
	/* Potential CSS injection */
	.highlight {
		background: var(--user-highlight-color, yellow);
	}
</style>