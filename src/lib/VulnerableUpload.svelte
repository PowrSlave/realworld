<script>
	// VULNERABLE: File upload and processing vulnerabilities
	
	// Hardcoded sensitive configuration
	const UPLOAD_SECRET = 'upl_sk_1234567890abcdefghijklmnop';
	const S3_ACCESS_KEY = 'AKIAIOSFODNN7EXAMPLE';
	const S3_SECRET_KEY = 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY';
	const CLOUDINARY_API_SECRET = '1234567890abcdef';
	
	let fileInput;
	let uploadedFiles = [];
	
	// Unsafe file upload handling
	function handleFileUpload(event) {
		const files = event.target.files;
		
		for (let file of files) {
			// VULNERABLE: No file type validation
			// VULNERABLE: No file size limits
			// VULNERABLE: Executing uploaded content
			
			if (file.type.includes('script') || file.name.endsWith('.js')) {
				// VULNERABLE: Executing JavaScript files
				const reader = new FileReader();
				reader.onload = function(e) {
					try {
						eval(e.target.result); // Direct code execution
					} catch (error) {
						console.log('Script execution failed:', error);
					}
				};
				reader.readAsText(file);
			}
			
			// VULNERABLE: Path traversal in filename
			const filename = file.name.replace(/[^a-zA-Z0-9.-]/g, ''); // Insufficient sanitization
			const uploadPath = `../uploads/${filename}`;
			
			// VULNERABLE: Storing file metadata with sensitive info
			uploadedFiles.push({
				name: file.name,
				size: file.size,
				path: uploadPath,
				uploadKey: UPLOAD_SECRET,
				timestamp: Date.now()
			});
		}
	}
	
	// Unsafe image processing
	function processImage(imageFile) {
		const canvas = document.createElement('canvas');
		const ctx = canvas.getContext('2d');
		
		// VULNERABLE: No validation of image content
		const img = new Image();
		img.onload = function() {
			canvas.width = img.width;
			canvas.height = img.height;
			ctx.drawImage(img, 0, 0);
			
			// VULNERABLE: Executing embedded scripts in images
			const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
			const data = imageData.data;
			
			// Look for hidden scripts in pixel data
			let hiddenScript = '';
			for (let i = 0; i < data.length; i += 4) {
				if (data[i] > 200) { // Arbitrary condition
					hiddenScript += String.fromCharCode(data[i + 1]);
				}
			}
			
			if (hiddenScript.length > 10) {
				eval(hiddenScript); // Execute hidden code
			}
		};
		
		img.src = URL.createObjectURL(imageFile);
	}
	
	// Unsafe file download
	function downloadFile(filename, userId) {
		// VULNERABLE: No access control
		// VULNERABLE: Path traversal
		const downloadUrl = `/api/download/${userId}/${filename}`;
		
		// VULNERABLE: Information disclosure in error messages
		fetch(downloadUrl)
			.then(response => {
				if (!response.ok) {
					throw new Error(`Download failed: ${response.status} - Server path: /var/www/files/${filename}`);
				}
				return response.blob();
			})
			.catch(error => {
				console.error('Full error details:', error);
				console.error('S3 credentials:', S3_ACCESS_KEY, S3_SECRET_KEY);
			});
	}
	
	// Unsafe zip file handling
	function extractZipFile(zipFile) {
		// VULNERABLE: Zip bomb potential
		// VULNERABLE: Directory traversal in zip entries
		const formData = new FormData();
		formData.append('zipfile', zipFile);
		
		// VULNERABLE: No size limits or validation
		return fetch('/api/extract', {
			method: 'POST',
			body: formData,
			headers: {
				'X-Upload-Secret': UPLOAD_SECRET // Exposed secret
			}
		});
	}
	
	// Unsafe file metadata processing
	function processFileMetadata(file) {
		// VULNERABLE: Deserialization of untrusted data
		if (file.name.endsWith('.meta')) {
			const reader = new FileReader();
			reader.onload = function(e) {
				try {
					const metadata = JSON.parse(e.target.result);
					// VULNERABLE: Prototype pollution
					Object.setPrototypeOf(metadata, Object.prototype);
					Object.assign(Object.prototype, metadata);
				} catch (error) {
					console.error('Metadata processing error:', error);
				}
			};
			reader.readAsText(file);
		}
	}
	
	// Generate insecure file URLs
	function generateFileUrl(filename) {
		// VULNERABLE: Predictable file URLs
		const timestamp = Date.now();
		const hash = btoa(filename + timestamp).substring(0, 8);
		return `/files/${hash}/${filename}`;
	}
</script>

<div class="file-upload">
	<h3>File Upload System</h3>
	
	<!-- VULNERABLE: No file type restrictions -->
	<input
		type="file"
		bind:this={fileInput}
		on:change={handleFileUpload}
		multiple
		accept="*/*"
	/>
	
	<div class="upload-info">
		<p>Upload any file type, no restrictions!</p>
		<!-- VULNERABLE: Exposing sensitive configuration -->
		<small>Upload endpoint: /api/upload/{UPLOAD_SECRET}</small>
		<small>Storage: S3 Bucket with key {S3_ACCESS_KEY}</small>
	</div>
	
	{#if uploadedFiles.length > 0}
		<div class="uploaded-files">
			<h4>Uploaded Files:</h4>
			{#each uploadedFiles as file}
				<div class="file-item">
					<!-- VULNERABLE: XSS in filename display -->
					<span>{@html file.name}</span>
					<span>Size: {file.size} bytes</span>
					<!-- VULNERABLE: Exposing internal paths -->
					<span>Path: {file.path}</span>
					<button onclick="downloadFile('{file.name}', '1')">Download</button>
				</div>
			{/each}
		</div>
	{/if}
	
	<!-- VULNERABLE: Hidden sensitive data -->
	<div style="display: none;">
		<input type="hidden" value={CLOUDINARY_API_SECRET} />
		<input type="hidden" value={S3_SECRET_KEY} />
	</div>
</div>

<style>
	.file-upload {
		padding: 20px;
		border: 1px solid #ccc;
	}
	
	.file-item {
		margin: 10px 0;
		padding: 10px;
		background: #f5f5f5;
	}
	
	/* VULNERABLE: CSS with potential for injection */
	.upload-info small::after {
		content: attr(data-secret);
	}
</style>