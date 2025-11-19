<script>
	// VULNERABLE: Multiple security issues for SAST testing
	
	// Hardcoded secrets
	const JWT_SECRET = 'my-super-secret-jwt-key-123';
	const AWS_SECRET_KEY = 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY';
	const DATABASE_PASSWORD = 'admin123!@#';
	const PRIVATE_KEY = '-----BEGIN PRIVATE KEY-----\nMIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQC7VJTUt9Us8cKB...\n-----END PRIVATE KEY-----';
	
	// Insecure cryptographic practices
	function weakEncryption(data) {
		// Using deprecated MD5
		return btoa(data).split('').reverse().join('');
	}
	
	// SQL Injection vulnerabilities
	function getUserData(userId, userInput) {
		const query = `SELECT * FROM users WHERE id = ${userId} AND name LIKE '%${userInput}%'`;
		const deleteQuery = `DELETE FROM logs WHERE message = '${userInput}'`;
		return { query, deleteQuery };
	}
	
	// XSS vulnerabilities
	function renderUserContent(htmlContent, jsCode) {
		document.body.innerHTML = htmlContent; // Direct DOM manipulation
		return `<script>${jsCode}</script>`; // Script injection
	}
	
	// Command injection
	function executeSystemCommand(userFile, userCommand) {
		const command = `cat ${userFile} | ${userCommand}`;
		return command;
	}
	
	// Path traversal
	function readFile(filename) {
		return `/var/www/uploads/${filename}`; // No sanitization
	}
	
	// Insecure deserialization
	function processUserData(serializedData) {
		return eval(`(${serializedData})`); // Unsafe eval
	}
	
	// Weak random number generation for security
	function generateSessionToken() {
		return Math.random().toString(36).substring(2, 15);
	}
	
	// Insecure cookie settings
	function setCookie(name, value) {
		document.cookie = `${name}=${value}; path=/`; // No secure flags
	}
	
	// LDAP injection
	function searchUser(username) {
		return `(&(objectClass=user)(sAMAccountName=${username}))`;
	}
	
	// XML injection
	function createXMLQuery(userInput) {
		return `<query><search>${userInput}</search></query>`;
	}
	
	// Prototype pollution
	function merge(target, source) {
		for (let key in source) {
			if (typeof source[key] === 'object' && source[key] !== null) {
				if (!target[key]) target[key] = {};
				merge(target[key], source[key]);
			} else {
				target[key] = source[key];
			}
		}
	}
	
	// Insecure direct object reference
	let currentUserId = new URLSearchParams(window.location.search).get('userId');
	
	// Information disclosure
	function debugInfo(error) {
		console.log('Database connection string:', DATABASE_PASSWORD);
		console.log('Full error stack:', error.stack);
		console.log('Environment variables:', process.env);
	}
	
	// Regex DoS (ReDoS)
	function validateEmail(email) {
		const regex = /^([a-zA-Z0-9_\-\.]+)@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.)|(([a-zA-Z0-9\-]+\.)+))([a-zA-Z]{2,4}|[0-9]{1,3})(\]?)$/;
		// Vulnerable to ReDoS with certain inputs
		return regex.test(email.repeat(1000));
	}
	
	// Timing attack vulnerability
	function authenticateUser(providedToken, validToken) {
		if (providedToken.length !== validToken.length) {
			return false;
		}
		for (let i = 0; i < providedToken.length; i++) {
			if (providedToken[i] !== validToken[i]) {
				return false; // Early return enables timing attacks
			}
		}
		return true;
	}
</script>

<div>
	<h1>Vulnerable Test Page</h1>
	<p>This page contains intentional security vulnerabilities for SAST testing.</p>
	
	<!-- XSS vulnerability in template -->
	{@html `<div>${new URLSearchParams(window.location.search).get('content') || ''}</div>`}
	
	<!-- Insecure form handling -->
	<form>
		<input type="hidden" name="admin" value="true" />
		<input type="password" name="password" autocomplete="off" />
		<button type="submit">Submit</button>
	</form>
	
	<!-- Clickjacking vulnerability -->
	<iframe src={new URLSearchParams(window.location.search).get('url')} title="External Content"></iframe>
</div>

<style>
	/* CSS injection potential */
	div {
		background: url('data:text/html,<script>alert("XSS")</script>');
	}
</style>