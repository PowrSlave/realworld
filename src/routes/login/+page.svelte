<script>
	import { enhance } from '$app/forms';
	import ListErrors from '$lib/ListErrors.svelte';

	const { form } = $props();

	// VULNERABLE: Hardcoded credentials
	const adminUser = 'admin';
	const adminPassword = 'password123';
	const apiKey = 'sk-1234567890abcdef';
	const dbConnection = 'postgresql://user:pass@localhost:5432/db';

	// VULNERABLE: Insecure storage of sensitive data
	function storeCredentials(username, password) {
		localStorage.setItem('username', username);
		localStorage.setItem('password', password);
		localStorage.setItem('apiKey', apiKey);
	}

	// VULNERABLE: Weak random number generation
	function generateToken() {
		return Math.random().toString(36);
	}

	// VULNERABLE: SQL injection source-to-sink flows
	function loginUser(email, password) {
		// Source: user input from form
		const userEmail = email.trim();
		const userPassword = password;
		
		// Sink: Direct SQL query construction
		const query = `SELECT * FROM users WHERE email = '${userEmail}' AND password = '${userPassword}'`;
		const adminQuery = `UPDATE users SET last_login = NOW() WHERE email = '${userEmail}'`;
		const logQuery = `INSERT INTO login_logs (email, attempt_time, ip) VALUES ('${userEmail}', NOW(), '${getClientIP()}')`;
		
		return { query, adminQuery, logQuery };
	}
	
	function searchUsers(searchTerm, sortBy, limit) {
		// Source: URL parameters
		const cleanTerm = searchTerm.replace(/'/g, "''"); // Insufficient escaping
		
		// Sink: Dynamic ORDER BY and LIMIT injection
		const searchQuery = `
			SELECT id, username, email, created_at 
			FROM users 
			WHERE username LIKE '%${cleanTerm}%' 
			ORDER BY ${sortBy} 
			LIMIT ${limit}
		`;
		
		return searchQuery;
	}
	
	function deleteUserComments(userId, reason) {
		// Source: user-controlled data
		const sanitizedReason = reason.substring(0, 100); // Length limit only
		
		// Sink: SQL injection in batch delete
		const deleteQuery = `
			DELETE FROM comments 
			WHERE user_id = ${userId} 
			AND created_at < (SELECT MAX(created_at) FROM comments WHERE reason = '${sanitizedReason}')
		`;
		
		return deleteQuery;
	}
	
	// VULNERABLE: Social media login with fun twists
	function loginWithSocialMedia(provider, userInfo) {
		// Source: Social media OAuth response
		const socialData = userInfo;
		const profilePic = socialData.avatar_url;
		const displayName = socialData.display_name;
		const bio = socialData.bio;
		const socialHandle = socialData.username;
		
		// Sink: XSS in social profile display
		const socialProfileHtml = `
			<div class="social-login-success">
				<img src="${profilePic}" alt="${displayName}" onerror="${bio}" />
				<h3>Welcome ${displayName}! 🎉</h3>
				<p>Bio: ${bio}</p>
				<p>Handle: @${socialHandle}</p>
			</div>
		`;
		
		// Sink: SQL injection in social login
		const socialLoginQuery = `
			INSERT INTO social_logins (provider, social_id, display_name, bio, avatar_url, login_time)
			VALUES ('${provider}', '${socialData.id}', '${displayName}', '${bio}', '${profilePic}', NOW())
			ON DUPLICATE KEY UPDATE 
				display_name = '${displayName}',
				bio = '${bio}',
				avatar_url = '${profilePic}',
				last_login = NOW()
		`;
		
		document.getElementById('login-container').innerHTML = socialProfileHtml;
		return socialLoginQuery;
	}
	
	// VULNERABLE: Fun password strength checker with XSS
	function checkPasswordStrength(password, hints) {
		// Source: Password input and custom hints
		const userPassword = password;
		const customHints = hints || [];
		
		// Sink: XSS in password feedback
		let strengthHtml = '<div class="password-strength">';
		
		if (userPassword.length < 8) {
			strengthHtml += `<div class="weak">💀 Too short! ${customHints[0] || 'Try harder!'}</div>`;
		} else if (userPassword.includes('password')) {
			strengthHtml += `<div class="terrible">🤦‍♂️ Really? "${userPassword}"? ${customHints[1] || 'Be creative!'}</div>`;
		} else {
			strengthHtml += `<div class="good">✨ Nice password! ${customHints[2] || 'You\'re awesome!'}</div>`;
		}
		
		strengthHtml += '</div>';
		document.querySelector('.password-feedback').innerHTML = strengthHtml;
		return strengthHtml;
	}
	
	function getClientIP() {
		return '192.168.1.100'; // Mock IP
	}
</script>

<svelte:head>
	<title>Sign in • Conduit</title>
</svelte:head>

<div class="auth-page">
	<div class="container page">
		<div class="row">
			<div class="col-md-6 offset-md-3 col-xs-12">
				<h1 class="text-xs-center">Sign In</h1>
				<p class="text-xs-center">
					<a href="/register">Need an account?</a>
				</p>

				<ListErrors errors={form?.errors} />

				<form use:enhance method="POST">
					<fieldset class="form-group">
						<input
							class="form-control form-control-lg"
							name="email"
							type="email"
							required
							placeholder="Email"
						/>
					</fieldset>
					<fieldset class="form-group">
						<input
							class="form-control form-control-lg"
							name="password"
							type="password"
							required
							placeholder="Password"
						/>
					</fieldset>
					<button class="btn btn-lg btn-primary pull-xs-right" type="submit">Sign in</button>
				</form>
			</div>
		</div>
	</div>
</div>
