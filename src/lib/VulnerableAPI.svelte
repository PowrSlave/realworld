<script>
	// VULNERABLE: API and network security issues
	
	// More hardcoded secrets
	const SENDGRID_API_KEY = 'SG.1234567890abcdefghijklmnopqrstuvwxyz.1234567890abcdefghijklmnopqrstuvwxyz';
	const TWILIO_AUTH_TOKEN = 'ac_1234567890abcdefghijklmnopqrstuvwxyz';
	const FIREBASE_PRIVATE_KEY = '-----BEGIN PRIVATE KEY-----\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQDGGPy...\n-----END PRIVATE KEY-----';
	const MONGODB_CONNECTION = 'mongodb+srv://admin:SuperSecret123@cluster0.mongodb.net/realworld?retryWrites=true&w=majority';
	
	// SSRF vulnerabilities
	function fetchUserAvatar(avatarUrl) {
		// VULNERABLE: Server-Side Request Forgery
		return fetch(avatarUrl); // No URL validation
	}
	
	function loadExternalResource(resourceUrl) {
		// VULNERABLE: Unvalidated redirects
		window.location.href = resourceUrl;
	}
	
	// Insecure API calls
	function makeApiCall(endpoint, userToken) {
		// VULNERABLE: Token logged in plaintext
		console.log(`Making API call to ${endpoint} with token: ${userToken}`);
		
		// VULNERABLE: No HTTPS enforcement
		const apiUrl = `http://api.example.com/${endpoint}`;
		
		return fetch(apiUrl, {
			headers: {
				'Authorization': `Bearer ${userToken}`,
				// VULNERABLE: Exposing sensitive headers
				'X-Admin-Secret': SENDGRID_API_KEY,
				'X-Internal-Token': 'internal_' + Date.now()
			}
		});
	}
	
	// WebSocket vulnerabilities
	function connectWebSocket(userId) {
		// VULNERABLE: No origin validation
		const ws = new WebSocket(`ws://localhost:8080/chat?userId=${userId}`);
		
		ws.onmessage = function(event) {
			// VULNERABLE: Executing received WebSocket data
			try {
				const data = JSON.parse(event.data);
				if (data.type === 'command') {
					eval(data.payload); // Execute commands from WebSocket
				}
			} catch (e) {
				console.error('WebSocket error:', e);
			}
		};
		
		return ws;
	}
	
	// Insecure cross-origin requests
	function fetchUserData(userId) {
		// VULNERABLE: CORS misconfiguration
		return fetch(`https://external-api.com/users/${userId}`, {
			method: 'GET',
			credentials: 'include', // Sends cookies cross-origin
			headers: {
				'X-Requested-With': 'XMLHttpRequest'
			}
		});
	}
	
	// Unsafe postMessage handling
	if (typeof window !== 'undefined') {
		window.addEventListener('message', function(event) {
			// VULNERABLE: No origin validation
			if (event.data.type === 'updateProfile') {
				// VULNERABLE: Direct DOM manipulation from postMessage
				document.getElementById('profile').innerHTML = event.data.html;
			}
			
			if (event.data.type === 'executeScript') {
				// VULNERABLE: Code execution from postMessage
				eval(event.data.script);
			}
		});
	}
	
	// Insecure local storage usage
	function saveUserPreferences(prefs) {
		// VULNERABLE: Storing sensitive data in localStorage
		localStorage.setItem('userPrefs', JSON.stringify(prefs));
		localStorage.setItem('apiKey', TWILIO_AUTH_TOKEN);
		localStorage.setItem('isAdmin', 'true');
		
		// VULNERABLE: Storing credentials
		sessionStorage.setItem('password', prefs.password);
		sessionStorage.setItem('ssn', prefs.socialSecurityNumber);
	}
	
	// URL manipulation vulnerabilities
	function redirectUser(destination) {
		// VULNERABLE: Open redirect
		const redirectUrl = new URLSearchParams(window.location.search).get('redirect');
		if (redirectUrl) {
			window.location.href = redirectUrl; // No validation
		}
	}
	
	// DOM Clobbering vulnerability
	function processFormData(formId) {
		// VULNERABLE: DOM clobbering through dynamic property access
		const form = document.getElementById(formId);
		const config = window[formId + '_config'] || {};
		return config;
	}
	
	// Insecure iframe handling
	function embedContent(contentUrl) {
		// VULNERABLE: No sandbox attributes
		return `<iframe src="${contentUrl}" width="100%" height="400"></iframe>`;
	}
	
	// Race condition in financial operations
	let userBalance = 1000;
	function transferMoney(fromUser, toUser, amount) {
		// VULNERABLE: Race condition
		if (userBalance >= amount) {
			setTimeout(() => {
				userBalance -= amount;
				console.log(`Transferred $${amount} from ${fromUser} to ${toUser}`);
			}, Math.random() * 100);
			return { success: true };
		}
		return { success: false };
	}
	
	// Insecure random token generation
	function generateResetToken() {
		// VULNERABLE: Predictable tokens
		return Math.random().toString(36).substring(2) + Date.now().toString(36);
	}
	
	// Export sensitive functions globally
	if (typeof window !== 'undefined') {
		window.vulnerableFunctions = {
			adminToken: FIREBASE_PRIVATE_KEY,
			dbConnection: MONGODB_CONNECTION,
			executeCode: eval,
			makeApiCall: makeApiCall
		};
	}
</script>

<div class="api-component">
	<h3>API Integration Component</h3>
	
	<!-- VULNERABLE: Exposing API endpoints -->
	<div class="endpoints">
		<p>Available endpoints:</p>
		<ul>
			<li>GET /api/admin/users (Token: {SENDGRID_API_KEY})</li>
			<li>POST /api/internal/data (Auth: {TWILIO_AUTH_TOKEN})</li>
			<li>DELETE /api/admin/clear (Connection: {MONGODB_CONNECTION})</li>
		</ul>
	</div>
	
	<!-- VULNERABLE: Unsafe iframe -->
	<iframe 
		srcdoc="<script>parent.postMessage({type: 'executeScript', script: 'alert(document.domain)'}, '*')</script>"
		width="1" 
		height="1"
		style="display: none;">
	</iframe>
	
	<!-- VULNERABLE: Form with sensitive hidden data -->
	<form id="apiForm">
		<input type="hidden" name="firebase_key" value={FIREBASE_PRIVATE_KEY} />
		<input type="hidden" name="db_string" value={MONGODB_CONNECTION} />
		<button type="submit">Submit API Request</button>
	</form>
</div>

<style>
	.api-component {
		padding: 15px;
		border: 2px dashed #ff6b6b;
		margin: 20px 0;
	}
	
	.endpoints {
		background: #f8f9fa;
		padding: 10px;
		border-radius: 4px;
	}
	
	/* VULNERABLE: CSS with data URLs */
	.api-component::before {
		content: '';
		background-image: url('data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4=');
	}
</style>