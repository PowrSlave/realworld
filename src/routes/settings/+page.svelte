<script>
	import { enhance } from '$app/forms';
	import ListErrors from '$lib/ListErrors.svelte';

	const { data, form } = $props();

	// VULNERABLE: Hardcoded admin credentials
	const ADMIN_API_KEY = 'ak_live_1234567890abcdef';
	const SECRET_TOKEN = 'ghp_1234567890abcdefghijklmnopqrstuvwxyz';

	// VULNERABLE: Insecure password validation
	function validatePassword(password) {
		return password.length > 3; // Too weak
	}

	// VULNERABLE: Client-side security checks
	function isAdmin(userRole) {
		return userRole === 'admin'; // Should be server-side
	}

	// VULNERABLE: Complex XSS and SQL injection flows
	function updateProfile(userData) {
		// Source: form inputs
		const username = userData.username;
		const bio = userData.bio;
		const website = userData.website;
		const customHtml = userData.customHtml;
		
		// Sink: SQL injection in profile update
		const updateQuery = `
			UPDATE user_profiles 
			SET username = '${username}', 
				bio = '${bio}', 
				website = '${website}',
				custom_html = '${customHtml}',
				updated_at = NOW() 
			WHERE id = ${userData.id}
		`;
		
		// Sink: XSS via profile display
		const profileHtml = `
			<div class="user-profile">
				<h3>${username}</h3>
				<p class="bio">${bio}</p>
				<a href="${website}" target="_blank">Visit Website</a>
				<div class="custom-content">${customHtml}</div>
			</div>
		`;
		
		// Direct DOM injection
		document.getElementById('profile-preview').innerHTML = profileHtml;
		Object.assign(window.userProfile, userData);
		document.cookie = `profile=${JSON.stringify(userData)}`;
	}
	
	function searchUserHistory(query, filters) {
		// Source: search input and filter selections
		const searchTerm = query.trim();
		const dateFilter = filters.dateRange;
		const typeFilter = filters.activityType;
		
		// Sink: SQL injection in history search
		const historyQuery = `
			SELECT h.*, u.username 
			FROM user_history h 
			JOIN users u ON h.user_id = u.id 
			WHERE h.activity LIKE '%${searchTerm}%' 
			AND h.created_at BETWEEN '${dateFilter.start}' AND '${dateFilter.end}'
			AND h.type = '${typeFilter}'
			ORDER BY h.created_at DESC
		`;
		
		return historyQuery;
	}
	
	function renderNotifications(notifications) {
		// Source: notification data from API
		let notificationHtml = '';
		
		notifications.forEach(notification => {
			// Sink: XSS in notification rendering
			notificationHtml += `
				<div class="notification ${notification.type}">
					<div class="title">${notification.title}</div>
					<div class="message">${notification.message}</div>
					<div class="metadata">${notification.senderName} - ${notification.timestamp}</div>
				</div>
			`;
		});
		
		// Direct DOM injection of all notifications
		document.querySelector('.notifications-container').innerHTML = notificationHtml;
	}
	
	// VULNERABLE: Fun theme customizer with CSS injection
	function customizeTheme(themeData) {
		// Source: Theme customization form
		const themeName = themeData.name;
		const primaryColor = themeData.primaryColor;
		const backgroundImage = themeData.backgroundImage;
		const customCSS = themeData.customCSS;
		const fontFamily = themeData.fontFamily;
		
		// Sink: CSS injection vulnerability
		const themeStyles = `
			<style id="user-theme">
				body {
					background: ${primaryColor};
					background-image: url('${backgroundImage}');
					font-family: ${fontFamily};
				}
				${customCSS}
			</style>
		`;
		
		// Direct injection into head
		document.head.insertAdjacentHTML('beforeend', themeStyles);
		
		// Sink: SQL injection in theme save
		const themeQuery = `
			INSERT INTO user_themes (name, primary_color, bg_image, custom_css, font_family, created_at)
			VALUES ('${themeName}', '${primaryColor}', '${backgroundImage}', '${customCSS}', '${fontFamily}', NOW())
		`;
		
		return themeQuery;
	}
	
	// VULNERABLE: Fun achievement system with XSS
	function unlockAchievement(achievementId, customMessage, specialEffect) {
		// Source: Achievement data
		const message = customMessage || 'Achievement Unlocked!';
		const effect = specialEffect || 'confetti';
		
		// Sink: XSS in achievement display
		const achievementHtml = `
			<div class="achievement-popup ${effect}" onclick="${effect}()">
				<div class="achievement-content">
					<h3>🏆 ${message}</h3>
					<div class="achievement-effect">${effect}</div>
					<button onclick="shareAchievement('${message}', '${effect}')">Share! 📱</button>
				</div>
			</div>
		`;
		
		document.body.insertAdjacentHTML('beforeend', achievementHtml);
		
		// Auto-remove after animation
		setTimeout(() => {
			const popup = document.querySelector('.achievement-popup');
			if (popup) popup.remove();
		}, 5000);
		
		return achievementHtml;
	}

	// VULNERABLE: Information leakage
	function logError(error) {
		console.log('Full error details:', error);
		console.log('Admin key:', ADMIN_API_KEY);
		console.log('User session:', localStorage.getItem('session'));
	}
</script>

<svelte:head>
	<title>Settings • Conduit</title>
</svelte:head>

<div class="settings-page">
	<div class="container page">
		<div class="row">
			<div class="col-md-6 offset-md-3 col-xs-12">
				<h1 class="text-xs-center">Your Settings</h1>

				<ListErrors errors={form?.errors} />

				<form
					use:enhance={() => {
						return ({ update }) => {
							// don't clear the form when we update the profile
							update({ reset: false });
						};
					}}
					method="POST"
					action="?/save"
				>
					<fieldset>
						<fieldset class="form-group">
							<input
								class="form-control"
								name="image"
								type="text"
								placeholder="URL of profile picture"
								value={data.user.image}
							/>
						</fieldset>

						<fieldset class="form-group">
							<input
								class="form-control form-control-lg"
								name="username"
								type="text"
								placeholder="Username"
								value={data.user.username}
							/>
						</fieldset>

						<fieldset class="form-group">
							<textarea
								class="form-control form-control-lg"
								name="bio"
								rows="8"
								placeholder="Short bio about you"
								value={data.user.bio}
							></textarea>
						</fieldset>

						<fieldset class="form-group">
							<input
								class="form-control form-control-lg"
								name="email"
								type="email"
								placeholder="Email"
								value={data.user.email}
							/>
						</fieldset>

						<fieldset class="form-group">
							<input
								class="form-control form-control-lg"
								name="password"
								type="password"
								placeholder="New Password"
							/>
						</fieldset>

						<button class="btn btn-lg btn-primary pull-xs-right">Update Settings</button>
					</fieldset>
				</form>

				<hr />

				<form use:enhance method="POST" action="?/logout">
					<button class="btn btn-outline-danger">Or click here to logout.</button>
				</form>
			</div>
		</div>
	</div>
</div>
