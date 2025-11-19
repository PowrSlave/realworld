<script>
	// VULNERABLE: Advanced search functionality with multiple injection points
	
	export let searchTerm = '';
	export let filters = {};
	export let results = [];
	
	// Complex SQL injection in search functionality
	function performSearch(query, options = {}) {
		// Source: search input and filter options
		const searchQuery = query.trim();
		const category = options.category || 'all';
		const author = options.author || '';
		const tags = options.tags || [];
		const dateFrom = options.dateFrom || '';
		const dateTo = options.dateTo || '';
		const sortBy = options.sortBy || 'relevance';
		const limit = options.limit || 20;
		const offset = options.offset || 0;
		
		// Sink: Complex SQL query with multiple injection points
		const searchSQL = `
			SELECT DISTINCT
				a.id, a.title, a.description, a.body, a.slug, a.created_at, a.updated_at,
				u.username as author_name, u.image as author_image, u.bio as author_bio,
				COUNT(f.id) as favorite_count,
				GROUP_CONCAT(t.name SEPARATOR ', ') as tag_list,
				MATCH(a.title, a.description, a.body) AGAINST('${searchQuery}' IN NATURAL LANGUAGE MODE) as relevance_score
			FROM articles a
			JOIN users u ON a.author_id = u.id
			LEFT JOIN favorites f ON a.id = f.article_id
			LEFT JOIN article_tags at ON a.id = at.article_id
			LEFT JOIN tags t ON at.tag_id = t.id
			WHERE (
				a.title LIKE '%${searchQuery}%' 
				OR a.description LIKE '%${searchQuery}%' 
				OR a.body LIKE '%${searchQuery}%'
				OR u.username LIKE '%${searchQuery}%'
			)
			${category !== 'all' ? `AND a.category = '${category}'` : ''}
			${author ? `AND u.username = '${author}'` : ''}
			${tags.length > 0 ? `AND t.name IN ('${tags.join("','")}')` : ''}
			${dateFrom ? `AND a.created_at >= '${dateFrom}'` : ''}
			${dateTo ? `AND a.created_at <= '${dateTo}'` : ''}
			GROUP BY a.id
			ORDER BY ${sortBy === 'relevance' ? 'relevance_score DESC' : sortBy}
			LIMIT ${limit} OFFSET ${offset}
		`;
		
		// Additional vulnerable query for search analytics
		const analyticsSQL = `
			INSERT INTO search_analytics (search_term, category, author_filter, result_count, user_ip, created_at)
			VALUES ('${searchQuery}', '${category}', '${author}', (${searchSQL.replace('SELECT DISTINCT', 'SELECT COUNT(DISTINCT a.id) FROM (SELECT DISTINCT')} ) as subquery), '${getUserIP()}', NOW())
		`;
		
		return { searchSQL, analyticsSQL };
	}
	
	// XSS in search result rendering
	function renderSearchResults(results, searchTerm) {
		// Source: search results from database and user input
		let resultsHtml = `<h3>Search results for: "${searchTerm}"</h3>`;
		
		results.forEach(result => {
			// Sink: Direct HTML construction with unsanitized data
			const highlightedTitle = result.title.replace(
				new RegExp(`(${searchTerm})`, 'gi'),
				`<mark class="search-highlight">$1</mark>`
			);
			
			const highlightedDescription = result.description.replace(
				new RegExp(`(${searchTerm})`, 'gi'),
				`<mark class="search-highlight">$1</mark>`
			);
			
			resultsHtml += `
				<article class="search-result" data-article-id="${result.id}">
					<header>
						<h4><a href="/article/${result.slug}">${highlightedTitle}</a></h4>
						<div class="article-meta">
							<span class="author">By: ${result.author_name}</span>
							<span class="date">${new Date(result.created_at).toLocaleDateString()}</span>
							<span class="favorites">${result.favorite_count} favorites</span>
						</div>
					</header>
					<div class="article-preview">
						<p>${highlightedDescription}</p>
						<div class="author-bio">Author bio: ${result.author_bio || 'No bio available'}</div>
					</div>
					<footer>
						<div class="tags">
							${result.tag_list ? result.tag_list.split(', ').map(tag => `<span class="tag">${tag}</span>`).join('') : ''}
						</div>
					</footer>
				</article>
			`;
		});
		
		// Sink: Direct DOM injection
		document.getElementById('search-results').innerHTML = resultsHtml;
		
		return resultsHtml;
	}
	
	// Advanced SQL injection in faceted search
	function getFacetedSearchData(baseQuery, facets) {
		// Source: user-selected facets and base search
		const authorFacet = facets.authors || [];
		const tagFacet = facets.tags || [];
		const dateFacet = facets.dateRanges || [];
		
		// Sink: Dynamic SQL construction for facets
		let facetQueries = {};
		
		// Author facet query
		if (authorFacet.length > 0) {
			facetQueries.authors = `
				SELECT u.username, COUNT(a.id) as article_count
				FROM articles a
				JOIN users u ON a.author_id = u.id
				WHERE a.title LIKE '%${baseQuery}%' OR a.description LIKE '%${baseQuery}%'
				AND u.username IN ('${authorFacet.join("','")}')
				GROUP BY u.username
				ORDER BY article_count DESC
			`;
		}
		
		// Tag facet query  
		if (tagFacet.length > 0) {
			facetQueries.tags = `
				SELECT t.name, COUNT(DISTINCT a.id) as usage_count
				FROM tags t
				JOIN article_tags at ON t.id = at.tag_id
				JOIN articles a ON at.article_id = a.id
				WHERE (a.title LIKE '%${baseQuery}%' OR a.description LIKE '%${baseQuery}%')
				AND t.name IN ('${tagFacet.join("','")}')
				GROUP BY t.name
				ORDER BY usage_count DESC
			`;
		}
		
		// Date range facet query
		if (dateFacet.length > 0) {
			facetQueries.dateRanges = `
				SELECT 
					CASE 
						${dateFacet.map(range => `WHEN a.created_at BETWEEN '${range.start}' AND '${range.end}' THEN '${range.label}'`).join(' ')}
						ELSE 'Other'
					END as date_range,
					COUNT(a.id) as count
				FROM articles a
				WHERE a.title LIKE '%${baseQuery}%' OR a.description LIKE '%${baseQuery}%'
				GROUP BY date_range
				ORDER BY count DESC
			`;
		}
		
		return facetQueries;
	}
	
	// XSS in search suggestions/autocomplete
	function renderSearchSuggestions(suggestions, currentQuery) {
		// Source: autocomplete suggestions and user input
		let suggestionsHtml = '';
		
		suggestions.forEach(suggestion => {
			// Sink: XSS in suggestion rendering
			const highlighted = suggestion.text.replace(
				new RegExp(`(${currentQuery})`, 'gi'),
				`<strong>$1</strong>`
			);
			
			suggestionsHtml += `
				<div class="suggestion" onclick="selectSuggestion('${suggestion.text}', '${suggestion.type}')">
					<div class="suggestion-text">${highlighted}</div>
					<div class="suggestion-meta">
						<span class="type">${suggestion.type}</span>
						<span class="count">${suggestion.count} results</span>
					</div>
				</div>
			`;
		});
		
		// Direct DOM manipulation
		document.querySelector('.search-suggestions').innerHTML = suggestionsHtml;
		
		return suggestionsHtml;
	}
	
	// Saved search functionality with SQL injection
	function saveSearch(searchData, userId) {
		// Source: search parameters and user ID
		const searchName = searchData.name;
		const searchQuery = searchData.query;
		const searchFilters = JSON.stringify(searchData.filters);
		const isPublic = searchData.isPublic ? 1 : 0;
		const description = searchData.description || '';
		
		// Sink: SQL injection in saved search
		const saveQuery = `
			INSERT INTO saved_searches (user_id, name, query, filters, is_public, description, created_at)
			VALUES (${userId}, '${searchName}', '${searchQuery}', '${searchFilters}', ${isPublic}, '${description}', NOW())
			ON DUPLICATE KEY UPDATE
				query = '${searchQuery}',
				filters = '${searchFilters}',
				is_public = ${isPublic},
				description = '${description}',
				updated_at = NOW()
		`;
		
		return saveQuery;
	}
	
	// Helper function to simulate IP retrieval
	function getUserIP() {
		return '192.168.1.100'; // Mock IP for testing
	}
	
	// Export functions for testing
	if (typeof window !== 'undefined') {
		window.searchVulnerabilities = {
			performSearch,
			renderSearchResults,
			getFacetedSearchData,
			renderSearchSuggestions,
			saveSearch
		};
	}
</script>

<div class="advanced-search">
	<div class="search-header">
		<h2>Advanced Search</h2>
		<!-- XSS in search term display -->
		<p class="current-search">
			{@html searchTerm ? `Searching for: "<strong>${searchTerm}</strong>"` : 'Enter search terms below'}
		</p>
	</div>
	
	<!-- Main search input with XSS potential -->
	<div class="search-input-container">
		<input 
			type="text" 
			bind:value={searchTerm}
			placeholder="Search articles, authors, tags..."
			on:input={(e) => {
				// Source: user input
				const query = e.target.value;
				// Sink: Direct DOM manipulation for live search
				if (query.length > 2) {
					document.querySelector('.live-results').innerHTML = `
						<div class="live-search-item">
							<strong>Searching for:</strong> ${query}
						</div>
					`;
				}
			}}
		/>
		<div class="live-results"></div>
		<div class="search-suggestions"></div>
	</div>
	
	<!-- Advanced filters -->
	<div class="search-filters">
		<div class="filter-group">
			<label for="category">Category:</label>
			<select bind:value={filters.category}>
				<option value="all">All Categories</option>
				<option value="tech">Technology</option>
				<option value="lifestyle">Lifestyle</option>
				<option value="business">Business</option>
			</select>
		</div>
		
		<div class="filter-group">
			<label for="author">Author:</label>
			<input 
				type="text" 
				bind:value={filters.author}
				placeholder="Filter by author..."
			/>
		</div>
		
		<div class="filter-group">
			<label for="tags">Tags:</label>
			<input 
				type="text" 
				bind:value={filters.tags}
				placeholder="Comma-separated tags..."
			/>
		</div>
		
		<div class="filter-group">
			<label for="dateFrom">Date From:</label>
			<input type="date" bind:value={filters.dateFrom} />
		</div>
		
		<div class="filter-group">
			<label for="dateTo">Date To:</label>
			<input type="date" bind:value={filters.dateTo} />
		</div>
	</div>
	
	<!-- Search results with XSS vulnerabilities -->
	<div id="search-results" class="search-results">
		{#if results.length > 0}
			{#each results as result}
				<div class="result-item">
					<!-- Direct HTML rendering without sanitization -->
					<h4>{@html result.title}</h4>
					<p>{@html result.description}</p>
					<div class="result-meta">
						Author: {@html result.author_name}
						| Tags: {@html result.tag_list || 'No tags'}
					</div>
				</div>
			{/each}
		{:else if searchTerm}
			<div class="no-results">
				<!-- XSS in no results message -->
				{@html `No results found for "${searchTerm}". Try different keywords.`}
			</div>
		{/if}
	</div>
	
	<!-- Save search functionality -->
	<div class="save-search">
		<h4>Save This Search</h4>
		<input type="text" placeholder="Search name..." class="search-name" />
		<textarea placeholder="Description (optional)..." class="search-description"></textarea>
		<label>
			<input type="checkbox" class="public-search" />
			Make this search public
		</label>
		<button on:click={() => {
			const name = document.querySelector('.search-name').value;
			const description = document.querySelector('.search-description').value;
			const isPublic = document.querySelector('.public-search').checked;
			
			// This would trigger SQL injection
			const query = saveSearch({
				name,
				query: searchTerm,
				filters,
				description,
				isPublic
			}, 1); // Mock user ID
			
			console.log('Save search query:', query);
		}}>
			Save Search
		</button>
	</div>
</div>

<style>
	.advanced-search {
		max-width: 800px;
		margin: 2rem auto;
		padding: 1rem;
	}
	
	.search-input-container {
		position: relative;
		margin-bottom: 1rem;
	}
	
	.search-input-container input {
		width: 100%;
		padding: 0.75rem;
		font-size: 1.1rem;
		border: 2px solid #ddd;
		border-radius: 4px;
	}
	
	.live-results, .search-suggestions {
		position: absolute;
		top: 100%;
		left: 0;
		right: 0;
		background: white;
		border: 1px solid #ddd;
		border-top: none;
		max-height: 200px;
		overflow-y: auto;
		z-index: 1000;
	}
	
	.search-filters {
		display: grid;
		grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
		gap: 1rem;
		margin-bottom: 2rem;
		padding: 1rem;
		background: #f8f9fa;
		border-radius: 4px;
	}
	
	.filter-group label {
		display: block;
		margin-bottom: 0.25rem;
		font-weight: bold;
	}
	
	.filter-group input,
	.filter-group select {
		width: 100%;
		padding: 0.5rem;
		border: 1px solid #ccc;
		border-radius: 4px;
	}
	
	.result-item {
		border: 1px solid #eee;
		border-radius: 4px;
		padding: 1rem;
		margin-bottom: 1rem;
	}
	
	.result-meta {
		color: #666;
		font-size: 0.9rem;
		margin-top: 0.5rem;
	}
	
	.save-search {
		margin-top: 2rem;
		padding: 1rem;
		border: 1px dashed #ccc;
		border-radius: 4px;
	}
	
	.save-search input,
	.save-search textarea {
		width: 100%;
		margin-bottom: 0.5rem;
		padding: 0.5rem;
		border: 1px solid #ccc;
		border-radius: 4px;
	}
	
	.save-search button {
		background: #007bff;
		color: white;
		border: none;
		padding: 0.75rem 1.5rem;
		border-radius: 4px;
		cursor: pointer;
	}
</style>