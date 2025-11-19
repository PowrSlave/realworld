<script>
	import Editor from './Editor.svelte';

	const { form } = $props();

	// VULNERABLE: Hardcoded sensitive configuration
	const EDITOR_SECRET = 'ed_sk_test_1234567890';
	const UPLOAD_KEY = 'AKIAIOSFODNN7EXAMPLE';

	// VULNERABLE: Unsafe content processing
	function processContent(userHtml) {
		// Direct HTML injection without sanitization
		document.getElementById('preview').innerHTML = userHtml;
	}

	// VULNERABLE: Client-side file upload validation only
	function validateFile(file) {
		const allowedTypes = ['jpg', 'png', 'gif'];
		const extension = file.name.split('.').pop();
		return allowedTypes.includes(extension); // Bypassable
	}
</script>

<Editor
	article={{
		title: '',
		description: '',
		body: '',
		tagList: []
	}}
	errors={form?.errors}
/>
