<script lang="ts">
	import { onMount } from 'svelte';
	let image: File | null = null;
	let imageUrl: string | null = null;
	let question: string = '';
	let isDragging: boolean = false;
	let description: string | null = null;
	let isLoading: boolean = false;

	const MAX_WIDTH = 800;
	const MAX_HEIGHT = 800;

	const resizeImage = (file: File): Promise<File> => {
		return new Promise((resolve) => {
			const img = new Image();
			img.src = URL.createObjectURL(file);
			img.onload = () => {
				const canvas = document.createElement('canvas');
				let width = img.width;
				let height = img.height;

				if (width > height) {
					if (width > MAX_WIDTH) {
						height = Math.round((height * MAX_WIDTH) / width);
						width = MAX_WIDTH;
					}
				} else {
					if (height > MAX_HEIGHT) {
						width = Math.round((width * MAX_HEIGHT) / height);
						height = MAX_HEIGHT;
					}
				}

				canvas.width = width;
				canvas.height = height;
				const ctx = canvas.getContext('2d');
				ctx.drawImage(img, 0, 0, width, height);

				canvas.toBlob((blob) => {
					if (blob) {
						resolve(new File([blob], file.name, { type: file.type }));
					}
				}, file.type);
			};
		});
	};

	const handleFileUpload = async (event: Event) => {
		const target = event.target as HTMLInputElement;
		if (target.files && target.files.length > 0) {
			image = target.files[0];
			image = await resizeImage(image);
			imageUrl = URL.createObjectURL(image);
		}
	};

	const handleDrop = async (event: DragEvent) => {
		event.preventDefault();
		isDragging = false;
		if (event.dataTransfer && event.dataTransfer.files.length > 0) {
			image = event.dataTransfer.files[0];
			image = await resizeImage(image);
			imageUrl = URL.createObjectURL(image);
		}
	};

	const handleDragOver = (event: DragEvent) => {
		event.preventDefault();
		isDragging = true;
	};

	const handleDragLeave = () => {
		isDragging = false;
	};

	const handleKeyDown = (event: KeyboardEvent) => {
		if (event.key === 'Enter' || event.key === ' ') {
			event.preventDefault();
			document.getElementById('file-input')?.click();
		}
	};

	const submitForm = async () => {
		if (!image || !question) {
			alert('Please upload an image and ask a question.');
			return;
		}

		isLoading = true;
		const formData = new FormData();
		formData.append('image', image);
		formData.append('question', question);

		try {
			const response = await fetch('/api/ask', {
				method: 'POST',
				body: formData
			});
			type Result = {
				description: string;
			};
			const result: Result = await response.json();
			description = result.description; // Store the description from the response
		} catch (error) {
			console.error('Error submitting form:', error);
			description = 'An error occurred while processing your request.';
		} finally {
			isLoading = false;
		}
	};
</script>

<div class="container">
	<h1>Reliable CMS Describe an Image AI</h1>
	<div
		class="upload-area {isDragging ? 'dragging' : ''}"
		role="button"
		tabindex="0"
		on:drop={handleDrop}
		on:dragover={handleDragOver}
		on:dragleave={handleDragLeave}
		on:click={() => document.getElementById('file-input')?.click()}
		on:keydown={handleKeyDown}
		aria-label="Upload Area: Drag & Drop Image or Click to Upload"
	>
		<input
			type="file"
			accept="image/*"
			on:change={handleFileUpload}
			style="display: none;"
			id="file-input"
		/>
		{#if imageUrl}
			<img src={imageUrl} alt="Upload preview" />
		{:else}
			<label for="file-input">Drag & Drop Image or Click to Upload</label>
		{/if}
	</div>
	<input
		type="text"
		class="question-input"
		placeholder="Ask a question about the photo..."
		bind:value={question}
	/>
	<button class="submit-button" on:click={submitForm}>Submit</button>
	{#if isLoading}
		<div class="loading-indicator">Processing your request...</div>
	{/if}
	{#if description}
		<div class="description">{description}</div>
	{/if}
	<div class="footer">
		<p>
			Built with 🧡
		</p>
	</div>
</div>

<style>
	.container {
		display: flex;
		flex-direction: column;
		align-items: center;
		margin-top: 60px;
	}

	h1 {
		color: #0070f3;
	}

	.upload-area {
		width: 400px;
		height: 300px;
		border: 3px dashed #ccc;
		display: flex;
		justify-content: center;
		align-items: center;
		margin-bottom: 30px;
		transition: background-color 0.3s;
		cursor: pointer;
		font-size: 20px;
	}

	.upload-area.dragging {
		background-color: #e0e0e0;
	}

	.upload-area img {
		max-width: 100%;
		max-height: 100%;
		display: block;
	}

	.question-input {
		width: 400px;
		padding: 15px;
		margin-bottom: 30px;
		font-size: 18px;
	}

	.submit-button {
		padding: 15px 30px;
		background-color: #0070f3;
		color: white;
		border: none;
		border-radius: 5px;
		cursor: pointer;
		font-size: 18px;
	}

	.submit-button:hover {
		background-color: #005bb5;
	}

	.loading-indicator {
		margin-top: 30px;
		font-size: 18px;
		color: #0070f3;
	}

	.description {
		margin-top: 30px;
		font-size: 20px;
		font-weight: bold;
		padding: 20px;
		border: 3px solid #0070f3;
		border-radius: 5px;
		background-color: #e0f7fa;
		color: #0070f3;
		max-width: 80%;
		text-align: center;
	}

	.footer {
		margin-top: 60px;
		text-align: center;
		font-size: 16px;
		color: #555;
	}

	.footer p {
		margin: 10px 0;
	}

	.footer a {
		color: #0070f3;
		text-decoration: none;
	}

	.footer a:hover {
		text-decoration: underline;
	}
</style>
