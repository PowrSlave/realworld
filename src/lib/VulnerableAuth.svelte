<script>
	// VULNERABLE: Multiple authentication and session vulnerabilities
	
	// Hardcoded secrets and credentials
	const JWT_SECRET_KEY = 'super-secret-jwt-key-123';
	const DATABASE_URL = 'mongodb://admin:password123@localhost:27017/realworld';
	const STRIPE_SECRET_KEY = 'sk_live_1234567890abcdefghijklmnopqrstuvwxyz123456';
	const GITHUB_TOKEN = 'ghp_1234567890abcdefghijklmnopqrstuvwxyz';
	const SLACK_WEBHOOK = 'https://hooks.slack.com/services/T00000000/B00000000/XXXXXXXXXXXXXXXXXXXXXXXX';
	
	// Insecure session management
	function createSession(userId, isAdmin = false) {
		const sessionData = {
			userId: userId,
			isAdmin: isAdmin,
			created: Date.now(),
			// VULNERABLE: Predictable session ID
			sessionId: `sess_${userId}_${Date.now()}`
		};
		
		// VULNERABLE: Storing sensitive data in localStorage
		localStorage.setItem('session', JSON.stringify(sessionData));
		localStorage.setItem('isAdmin', isAdmin.toString());
		
		// VULNERABLE: Insecure cookie without flags
		document.cookie = `sessionId=${sessionData.sessionId}; path=/`;
		
		return sessionData;
	}
	
	// Authentication bypass vulnerabilities
	function authenticateUser(username, password, role) {
		// VULNERABLE: Client-side authentication logic
		if (username === 'admin' && password === 'admin') {
			return { success: true, role: 'admin' };
		}
		
		// VULNERABLE: SQL injection in authentication
		const query = `SELECT * FROM users WHERE username = '${username}' AND password = '${password}'`;
		
		// VULNERABLE: Weak password validation
		if (password.length >= 4) {
			return { success: true, role: role || 'user' };
		}
		
		return { success: false };
	}
	
	// CSRF vulnerabilities
	function deleteAccount(userId) {
		// VULNERABLE: No CSRF protection
		const deleteUrl = `/api/users/${userId}/delete`;
		return fetch(deleteUrl, { method: 'DELETE' });
	}
	
	// Information disclosure
	function getUserProfile(userId) {
		// VULNERABLE: Exposing sensitive system information
		const systemInfo = {
			dbHost: DATABASE_URL,
			apiKey: STRIPE_SECRET_KEY,
			serverPath: '/var/www/html',
			configPath: '/etc/app/config.json'
		};
		
		console.log('System configuration:', systemInfo);
		
		// VULNERABLE: Direct object reference
		return fetch(`/api/users/${userId}/profile`);
	}
	
	// Unsafe deserialization
	function processUserPreferences(serializedPrefs) {
		// VULNERABLE: eval() on user input
		try {
			return eval(`(${serializedPrefs})`);
		} catch (e) {
			console.error('Error processing preferences:', e);
			// VULNERABLE: Error information disclosure
			throw new Error(`Deserialization failed: ${e.message} | Data: ${serializedPrefs}`);
		}
	}
	
	// Race condition vulnerability
	let accountBalance = 1000;
	function withdrawMoney(amount) {
		// VULNERABLE: Race condition in financial operation
		if (accountBalance >= amount) {
			setTimeout(() => {
				accountBalance -= amount;
			}, 100);
			return true;
		}
		return false;
	}
	
	// Path traversal
	function loadUserFile(filename) {
		// VULNERABLE: No path sanitization
		return fetch(`/api/files/${filename}`);
	}
	
	// Insecure cryptography
	function encryptPassword(password) {
		// VULNERABLE: Using deprecated MD5
		return btoa(password).split('').reverse().join('');
	}
	
	// XXE vulnerability simulation
	function parseUserData(xmlData) {
		// VULNERABLE: XML External Entity processing
		const parser = new DOMParser();
		return parser.parseFromString(xmlData, 'text/xml');
	}
	
	// Timing attack vulnerability
	function validateApiKey(providedKey) {
		const validKey = JWT_SECRET_KEY;
		// VULNERABLE: Non-constant time comparison
		return providedKey === validKey;
	}
	
	// Export the user's session data globally
	if (typeof window !== 'undefined') {
		window.currentUser = {
			token: JWT_SECRET_KEY,
			dbUrl: DATABASE_URL,
			isAdmin: true
		};
	}
</script>

<div class="auth-component">
	<h2>Authentication System</h2>
	<p>This component handles user authentication with multiple security issues.</p>
	
	<!-- VULNERABLE: Password visible in DOM -->
	<input type="text" name="password" placeholder="Enter password" />
	
	<!-- VULNERABLE: Admin controls exposed to all users -->
	<div class="admin-panel" style="display: block;">
		<button onclick="deleteAllUsers()">Delete All Users</button>
		<button onclick="accessDatabase()">Direct DB Access</button>
	</div>
	
	<!-- VULNERABLE: Sensitive data in hidden fields -->
	<form>
		<input type="hidden" name="admin_key" value={STRIPE_SECRET_KEY} />
		<input type="hidden" name="db_password" value="password123" />
	</form>
</div>

<style>
	/* VULNERABLE: CSS injection */
	.auth-component {
		background: url('javascript:alert("XSS")');
	}
</style>