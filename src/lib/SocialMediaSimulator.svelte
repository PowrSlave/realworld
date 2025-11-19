<script>
	// VULNERABLE: Fun social media simulator with tons of XSS and SQL injection! 📱
	
	export let posts = [];
	export let currentUser = 'AwesomeUser123';
	
	// VULNERABLE: Post creation with multiple injection points
	function createSocialPost(content, hashtags, mentions, mediaUrl, mood) {
		// Source: User input from post form
		const postContent = content;
		const postHashtags = hashtags.split(' ').filter(tag => tag.startsWith('#'));
		const postMentions = mentions.split(' ').filter(mention => mention.startsWith('@'));
		const postMedia = mediaUrl || '';
		const postMood = mood || '😊';
		
		// Sink: XSS in post display
		const postHtml = `
			<div class="social-post" data-post-id="${Date.now()}">
				<div class="post-header">
					<div class="user-info">
						<img src="/avatars/${currentUser}.jpg" alt="${currentUser}" onerror="${postMood}" />
						<span class="username" onclick="viewProfile('${currentUser}')">${currentUser}</span>
						<span class="mood-indicator">${postMood}</span>
					</div>
					<div class="post-timestamp">${new Date().toLocaleString()}</div>
				</div>
				<div class="post-content">
					<p>${postContent}</p>
					${postMedia ? `<img src="${postMedia}" alt="Post media" onclick="${postContent}" />` : ''}
				</div>
				<div class="post-hashtags">
					${postHashtags.map(tag => `<span class="hashtag" onclick="searchHashtag('${tag}')">${tag}</span>`).join(' ')}
				</div>
				<div class="post-mentions">
					${postMentions.map(mention => `<span class="mention" onclick="viewProfile('${mention.substring(1)}')">${mention}</span>`).join(' ')}
				</div>
				<div class="post-actions">
					<button onclick="likePost('${postContent}', '${currentUser}')">❤️ Like</button>
					<button onclick="sharePost('${postContent}', '${postHashtags.join(' ')}')">🔄 Share</button>
					<button onclick="commentOnPost('${postContent}')">💬 Comment</button>
				</div>
			</div>
		`;
		
		document.getElementById('social-feed').insertAdjacentHTML('afterbegin', postHtml);
		
		// Sink: SQL injection in post storage
		const postQuery = `
			INSERT INTO social_posts (user_id, content, hashtags, mentions, media_url, mood, created_at)
			VALUES (
				(SELECT id FROM users WHERE username = '${currentUser}'),
				'${postContent}',
				'${postHashtags.join(',')}',
				'${postMentions.join(',')}',
				'${postMedia}',
				'${postMood}',
				NOW()
			)
		`;
		
		return postQuery;
	}
	
	// VULNERABLE: Story feature with XSS
	function addStoryContent(storyType, content, duration, effects) {
		// Source: Story creation inputs
		const type = storyType; // 'text', 'image', 'video'
		const storyContent = content;
		const storyDuration = duration || 24; // hours
		const visualEffects = effects || [];
		
		// Sink: XSS in story display
		const storyHtml = `
			<div class="story-item ${type}" onclick="viewFullStory('${storyContent}', '${effects}')">
				<div class="story-preview">
					${type === 'text' ? 
						`<div class="story-text">${storyContent}</div>` : 
						`<img src="${storyContent}" alt="Story" onload="${effects.join(';')}" />`
					}
				</div>
				<div class="story-effects">
					${visualEffects.map(effect => `<span class="effect">${effect}</span>`).join('')}
				</div>
				<div class="story-meta">
					<span class="story-user">${currentUser}</span>
					<span class="story-duration">${storyDuration}h</span>
				</div>
			</div>
		`;
		
		document.querySelector('.stories-container').innerHTML += storyHtml;
		
		// Sink: SQL injection in story storage
		const storyQuery = `
			INSERT INTO user_stories (username, content, story_type, duration, effects, created_at, expires_at)
			VALUES (
				'${currentUser}',
				'${storyContent}',
				'${type}',
				${storyDuration},
				'${visualEffects.join(',')}',
				NOW(),
				DATE_ADD(NOW(), INTERVAL ${storyDuration} HOUR)
			)
		`;
		
		return storyQuery;
	}
	
	// VULNERABLE: Live chat with XSS and command injection
	function sendLiveMessage(message, recipientUser, messageType) {
		// Source: Chat input and recipient
		const chatMessage = message;
		const recipient = recipientUser;
		const msgType = messageType || 'text';
		
		// Special commands in chat
		if (chatMessage.startsWith('/')) {
			const command = chatMessage.substring(1).split(' ')[0];
			const args = chatMessage.substring(1).split(' ').slice(1);
			
			// Sink: Command injection vulnerability
			if (command === 'gif') {
				const gifUrl = args[0];
				const gifHtml = `<img src="${gifUrl}" alt="GIF" onclick="${args.join(' ')}" />`;
				return sendChatCommand('gif', gifHtml, recipient);
			} else if (command === 'poll') {
				const pollQuestion = args.join(' ');
				return createInstantPoll(pollQuestion, recipient);
			}
		}
		
		// Sink: XSS in message display
		const messageHtml = `
			<div class="chat-message ${msgType}">
				<div class="message-sender">${currentUser}</div>
				<div class="message-content">${chatMessage}</div>
				<div class="message-timestamp">${new Date().toLocaleTimeString()}</div>
				<div class="message-actions">
					<button onclick="reactToMessage('${chatMessage}', '❤️')">❤️</button>
					<button onclick="reactToMessage('${chatMessage}', '😂')">😂</button>
					<button onclick="forwardMessage('${chatMessage}', '${recipient}')">➡️</button>
				</div>
			</div>
		`;
		
		// Update chat for both users
		document.querySelector(`#chat-${recipient}`).innerHTML += messageHtml;
		
		// Sink: SQL injection in message storage
		const messageQuery = `
			INSERT INTO chat_messages (sender, recipient, message, message_type, sent_at)
			VALUES ('${currentUser}', '${recipient}', '${chatMessage}', '${msgType}', NOW())
		`;
		
		return messageQuery;
	}
	
	// VULNERABLE: Trend analysis with XSS
	function generateTrendingTopics(userInterests, customTopics) {
		// Source: User interests and custom topics
		const interests = userInterests || [];
		const custom = customTopics || [];
		
		// Mock trending calculation with user input
		const trendingHtml = `
			<div class="trending-section">
				<h3>🔥 Trending Now 🔥</h3>
				<div class="trending-topics">
					${interests.map(interest => 
						`<div class="trend-item" onclick="exploreTrend('${interest}')">
							#${interest} 
							<span class="trend-count">${Math.floor(Math.random() * 10000)} posts</span>
						</div>`
					).join('')}
					${custom.map(topic => 
						`<div class="custom-trend" onclick="customTrendAction('${topic}')">
							🌟 ${topic}
						</div>`
					).join('')}
				</div>
			</div>
		`;
		
		// Sink: SQL injection in trend tracking
		const trendQuery = `
			UPDATE trending_topics 
			SET post_count = post_count + 1,
				last_updated = NOW(),
				custom_data = '${JSON.stringify(custom)}'
			WHERE topic IN ('${interests.join("','")}')
		`;
		
		document.getElementById('trending-sidebar').innerHTML = trendingHtml;
		return trendQuery;
	}
	
	// VULNERABLE: User profile customization with CSS injection
	function customizeProfile(profileData) {
		// Source: Profile customization form
		const bio = profileData.bio;
		const customCSS = profileData.customCSS;
		const theme = profileData.theme;
		const backgroundMusic = profileData.backgroundMusic;
		const customHTML = profileData.customHTML;
		
		// Sink: CSS injection in profile styling
		const profileStyles = `
			<style id="user-profile-${currentUser}">
				.profile-${currentUser} {
					${customCSS}
				}
				.profile-background {
					background-image: url('${profileData.backgroundImage}');
				}
			</style>
		`;
		
		// Sink: XSS in profile display
		const profileHtml = `
			<div class="user-profile profile-${currentUser}">
				<div class="profile-header">
					<h2>${currentUser}</h2>
					<div class="profile-bio">${bio}</div>
				</div>
				<div class="profile-custom-content">
					${customHTML}
				</div>
				<div class="profile-background"></div>
				${backgroundMusic ? `<audio autoplay loop src="${backgroundMusic}"></audio>` : ''}
			</div>
		`;
		
		document.head.insertAdjacentHTML('beforeend', profileStyles);
		document.getElementById('profile-container').innerHTML = profileHtml;
		
		// Sink: SQL injection in profile update
		const profileQuery = `
			UPDATE user_profiles 
			SET bio = '${bio}',
				custom_css = '${customCSS}',
				theme = '${theme}',
				background_music = '${backgroundMusic}',
				custom_html = '${customHTML}',
				updated_at = NOW()
			WHERE username = '${currentUser}'
		`;
		
		return profileQuery;
	}
	
	// VULNERABLE: Notification system with XSS
	function sendNotification(notificationType, message, actionUrl) {
		// Source: Notification data
		const type = notificationType;
		const notifMessage = message;
		const actionLink = actionUrl;
		
		// Notification types with fun emojis
		const notificationIcons = {
			'like': '❤️',
			'comment': '💬',
			'follow': '👥',
			'mention': '📢',
			'share': '🔄',
			'achievement': '🏆',
			'system': '⚙️'
		};
		
		const icon = notificationIcons[type] || '📱';
		
		// Sink: XSS in notification display
		const notificationHtml = `
			<div class="notification ${type}" onclick="handleNotification('${actionLink}', '${notifMessage}')">
				<div class="notification-icon">${icon}</div>
				<div class="notification-content">
					<div class="notification-message">${notifMessage}</div>
					<div class="notification-action">
						<a href="${actionLink}" onclick="${notifMessage}">View Details</a>
					</div>
				</div>
				<div class="notification-timestamp">${new Date().toLocaleTimeString()}</div>
			</div>
		`;
		
		document.getElementById('notifications-panel').insertAdjacentHTML('afterbegin', notificationHtml);
		
		// Sink: SQL injection in notification logging
		const notificationQuery = `
			INSERT INTO user_notifications (user_id, type, message, action_url, created_at, is_read)
			VALUES (
				(SELECT id FROM users WHERE username = '${currentUser}'),
				'${type}',
				'${notifMessage}',
				'${actionLink}',
				NOW(),
				0
			)
		`;
		
		return notificationQuery;
	}
	
	// Export social media functions
	if (typeof window !== 'undefined') {
		window.socialMediaVulnerabilities = {
			createSocialPost,
			addStoryContent,
			sendLiveMessage,
			generateTrendingTopics,
			customizeProfile,
			sendNotification
		};
	}
</script>

<div class="social-media-simulator">
	<div class="social-header">
		<h2>📱 SOCIAL MEDIA SIMULATOR 📱</h2>
		<div class="user-status">
			<span>Logged in as: <strong>{currentUser}</strong></span>
			<span class="online-indicator">🟢 Online</span>
		</div>
	</div>
	
	<!-- Post creation form -->
	<div class="post-creator">
		<h3>✨ Create a Post ✨</h3>
		<textarea 
			placeholder="What's on your mind? Share something awesome! 🎉"
			rows="3"
			id="post-content">
		</textarea>
		<div class="post-options">
			<input type="text" placeholder="#hashtags #fun #social" id="post-hashtags" />
			<input type="text" placeholder="@mention @friends" id="post-mentions" />
			<input type="url" placeholder="Media URL (optional)" id="post-media" />
			<select id="post-mood">
				<option value="😊">😊 Happy</option>
				<option value="🤔">🤔 Thoughtful</option>
				<option value="🔥">🔥 Excited</option>
				<option value="😎">😎 Cool</option>
				<option value="🥳">🥳 Celebrating</option>
			</select>
		</div>
		<button onclick="
			const content = document.getElementById('post-content').value;
			const hashtags = document.getElementById('post-hashtags').value;
			const mentions = document.getElementById('post-mentions').value;
			const media = document.getElementById('post-media').value;
			const mood = document.getElementById('post-mood').value;
			createSocialPost(content, hashtags, mentions, media, mood);
		">🚀 Post It!</button>
	</div>
	
	<!-- Stories section -->
	<div class="stories-section">
		<h3>📖 Stories</h3>
		<div class="stories-container"></div>
		<div class="add-story">
			<button onclick="
				const content = prompt('Story content:');
				const effects = prompt('Visual effects (comma-separated):').split(',');
				addStoryContent('text', content, 24, effects);
			">➕ Add Story</button>
		</div>
	</div>
	
	<!-- Main feed -->
	<div class="social-feed-container">
		<div id="social-feed" class="social-feed">
			<div class="feed-placeholder">
				<p>🌟 Your awesome posts will appear here! 🌟</p>
			</div>
		</div>
		
		<!-- Trending sidebar -->
		<div id="trending-sidebar" class="trending-sidebar">
			<p>Loading trending topics...</p>
		</div>
	</div>
	
	<!-- Live chat section -->
	<div class="live-chat-section">
		<h3>💬 Live Chat</h3>
		<div class="chat-tabs">
			{#each ['Friend1', 'Friend2', 'GroupChat'] as chatUser}
				<div class="chat-tab" onclick="switchToChat('{chatUser}')">{chatUser}</div>
			{/each}
		</div>
		<div id="chat-Friend1" class="chat-window active">
			<div class="chat-messages"></div>
			<input 
				type="text" 
				placeholder="Type a message... (try /gif or /poll)" 
				on:keypress={(e) => {
					if (e.key === 'Enter') {
						sendLiveMessage(e.target.value, 'Friend1', 'text');
						e.target.value = '';
					}
				}}
			/>
		</div>
	</div>
	
	<!-- Notifications panel -->
	<div id="notifications-panel" class="notifications-panel">
		<h4>🔔 Notifications</h4>
	</div>
	
	<!-- Profile customization -->
	<div id="profile-container" class="profile-container">
		<h3>👤 Your Profile</h3>
		<button onclick="
			const bio = prompt('Enter your bio:');
			const customCSS = prompt('Custom CSS styles:');
			const theme = prompt('Theme name:');
			const bgMusic = prompt('Background music URL:');
			const customHTML = prompt('Custom HTML content:');
			customizeProfile({bio, customCSS, theme, backgroundMusic: bgMusic, customHTML});
		">🎨 Customize Profile</button>
	</div>
</div>

<style>
	.social-media-simulator {
		max-width: 1200px;
		margin: 2rem auto;
		padding: 1rem;
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		border-radius: 15px;
		color: white;
		font-family: 'Arial', sans-serif;
	}
	
	.social-header {
		text-align: center;
		margin-bottom: 2rem;
		padding: 1rem;
		background: rgba(255, 255, 255, 0.1);
		border-radius: 10px;
	}
	
	.user-status {
		margin-top: 0.5rem;
		display: flex;
		justify-content: center;
		gap: 1rem;
	}
	
	.post-creator {
		background: rgba(255, 255, 255, 0.1);
		padding: 1rem;
		border-radius: 10px;
		margin-bottom: 2rem;
	}
	
	.post-creator textarea {
		width: 100%;
		padding: 0.75rem;
		border-radius: 5px;
		border: none;
		margin-bottom: 1rem;
		resize: vertical;
	}
	
	.post-options {
		display: grid;
		grid-template-columns: 1fr 1fr 1fr 150px;
		gap: 0.5rem;
		margin-bottom: 1rem;
	}
	
	.post-options input, .post-options select {
		padding: 0.5rem;
		border-radius: 5px;
		border: none;
	}
	
	.post-creator button {
		background: #ff6b6b;
		color: white;
		border: none;
		padding: 0.75rem 1.5rem;
		border-radius: 25px;
		cursor: pointer;
		font-weight: bold;
		transition: all 0.3s ease;
	}
	
	.post-creator button:hover {
		background: #ff5252;
		transform: translateY(-2px);
	}
	
	.stories-section {
		margin-bottom: 2rem;
	}
	
	.stories-container {
		display: flex;
		gap: 1rem;
		overflow-x: auto;
		padding: 1rem 0;
	}
	
	.story-item {
		min-width: 120px;
		height: 200px;
		background: linear-gradient(45deg, #ff9a9e, #fecfef);
		border-radius: 10px;
		padding: 0.5rem;
		cursor: pointer;
		transition: transform 0.3s ease;
	}
	
	.story-item:hover {
		transform: scale(1.05);
	}
	
	.social-feed-container {
		display: grid;
		grid-template-columns: 2fr 1fr;
		gap: 2rem;
		margin-bottom: 2rem;
	}
	
	.social-feed {
		background: rgba(255, 255, 255, 0.1);
		border-radius: 10px;
		padding: 1rem;
		min-height: 400px;
	}
	
	.social-post {
		background: rgba(255, 255, 255, 0.2);
		border-radius: 10px;
		padding: 1rem;
		margin-bottom: 1rem;
		transition: transform 0.2s ease;
	}
	
	.social-post:hover {
		transform: translateY(-2px);
	}
	
	.post-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 0.5rem;
	}
	
	.user-info {
		display: flex;
		align-items: center;
		gap: 0.5rem;
	}
	
	.user-info img {
		width: 40px;
		height: 40px;
		border-radius: 50%;
	}
	
	.post-actions {
		display: flex;
		gap: 1rem;
		margin-top: 1rem;
	}
	
	.post-actions button {
		background: rgba(255, 255, 255, 0.2);
		color: white;
		border: none;
		padding: 0.5rem 1rem;
		border-radius: 20px;
		cursor: pointer;
		transition: all 0.2s ease;
	}
	
	.post-actions button:hover {
		background: rgba(255, 255, 255, 0.3);
	}
	
	.trending-sidebar {
		background: rgba(255, 255, 255, 0.1);
		border-radius: 10px;
		padding: 1rem;
	}
	
	.live-chat-section {
		background: rgba(255, 255, 255, 0.1);
		border-radius: 10px;
		padding: 1rem;
		margin-bottom: 2rem;
	}
	
	.chat-tabs {
		display: flex;
		gap: 0.5rem;
		margin-bottom: 1rem;
	}
	
	.chat-tab {
		background: rgba(255, 255, 255, 0.2);
		padding: 0.5rem 1rem;
		border-radius: 20px;
		cursor: pointer;
		transition: all 0.2s ease;
	}
	
	.chat-tab:hover {
		background: rgba(255, 255, 255, 0.3);
	}
	
	.chat-window {
		display: none;
	}
	
	.chat-window.active {
		display: block;
	}
	
	.chat-messages {
		height: 200px;
		overflow-y: auto;
		border: 1px solid rgba(255, 255, 255, 0.3);
		padding: 0.5rem;
		margin-bottom: 0.5rem;
		border-radius: 5px;
	}
	
	.chat-message {
		margin-bottom: 0.5rem;
		padding: 0.5rem;
		background: rgba(255, 255, 255, 0.1);
		border-radius: 5px;
	}
	
	.notifications-panel {
		background: rgba(255, 255, 255, 0.1);
		border-radius: 10px;
		padding: 1rem;
		max-height: 300px;
		overflow-y: auto;
	}
	
	.notification {
		display: flex;
		align-items: center;
		gap: 1rem;
		padding: 0.75rem;
		background: rgba(255, 255, 255, 0.1);
		border-radius: 8px;
		margin-bottom: 0.5rem;
		cursor: pointer;
		transition: all 0.2s ease;
	}
	
	.notification:hover {
		background: rgba(255, 255, 255, 0.2);
	}
	
	.hashtag, .mention {
		color: #4ecdc4;
		cursor: pointer;
		margin-right: 0.5rem;
	}
	
	.hashtag:hover, .mention:hover {
		text-decoration: underline;
	}
</style>