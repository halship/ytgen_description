<script lang="ts">
	import type { Tag } from '$lib/types';

	let tags = $state<Tag[]>([]);
	let output = $derived.by(() => {
		const tagsResult = tags.map((tag) => `#${tag.name}`).join(' ');
		return tagsResult;
	});

	function addTag() {
		tags = [...tags, { id: tags.length, name: '' }];
	}

	function removeTag(id: number) {
		tags = tags.filter((tag) => tag.id !== id);
	}

	let copied = $state(false);

	async function copyToClipboard() {
		try {
			await navigator.clipboard.writeText(output);
			copied = true;
			setTimeout(() => {
				copied = false;
			}, 2000);
		} catch (err) {
			console.error('コピーに失敗しました:', err);
		}
	}
</script>

<div class="container">
	<h1>Youtube説明文ジェネレータ</h1>

	<section class="section">
		<h2>タグ入力欄</h2>
		<div class="tag-input-container">
			{#each tags as tag (tag.id)}
				<div class="tag-row">
					<input type="text" bind:value={tag.name} placeholder="タグ名を入力" />
					<button class="btn btn-danger" onclick={() => removeTag(tag.id)}>削除</button>
				</div>
			{/each}
			<button class="btn btn-primary" onclick={addTag}>タグ追加</button>
		</div>
	</section>

	<section class="section">
		<h2>出力結果</h2>
		<div class="output-container">
			<textarea rows={10} cols={80} readonly>{output}</textarea>
			<div class="output-footer">
				<button class="btn btn-copy" onclick={copyToClipboard}>
					{copied ? 'コピーしました！' : '一括コピー'}
				</button>
			</div>
		</div>
	</section>
</div>

<style>
	.container {
		max-width: 800px;
		margin: 0 auto;
		padding: 2rem;
	}

	h1 {
		font-size: 2rem;
		font-weight: bold;
		margin-bottom: 2rem;
		color: #1a1a1a;
		text-align: center;
	}

	h2 {
		font-size: 1.5rem;
		font-weight: 600;
		margin-bottom: 1rem;
		color: #333;
	}

	.section {
		margin-bottom: 2.5rem;
	}

	.tag-input-container {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
	}

	.tag-row {
		display: flex;
		flex-direction: row;
		gap: 0.75rem;
		align-items: center;
	}

	input[type='text'] {
		flex: 1;
		padding: 0.75rem;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 1rem;
		transition:
			border-color 0.2s,
			box-shadow 0.2s;
		box-sizing: border-box;
	}

	input[type='text']:focus {
		outline: none;
		border-color: #4a90e2;
		box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
	}

	.btn {
		padding: 0.75rem 1.5rem;
		border: none;
		border-radius: 4px;
		font-size: 1rem;
		font-weight: 500;
		cursor: pointer;
		transition:
			background-color 0.2s,
			transform 0.1s;
	}

	.btn:hover {
		transform: translateY(-1px);
	}

	.btn:active {
		transform: translateY(0);
	}

	.btn-primary {
		background-color: #4a90e2;
		color: white;
	}

	.btn-primary:hover {
		background-color: #357abd;
	}

	.btn-danger {
		background-color: #e74c3c;
		color: white;
		padding: 0.75rem 1rem;
	}

	.btn-danger:hover {
		background-color: #c0392b;
	}

	.output-container {
		width: 100%;
	}

	.output-footer {
		display: flex;
		justify-content: flex-end;
		margin-bottom: 0.75rem;
	}

	.btn-copy {
		background-color: #27ae60;
		color: white;
	}

	.btn-copy:hover {
		background-color: #229954;
	}

	textarea {
		width: 100%;
		padding: 1rem;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 1rem;
		background-color: #f9f9f9;
		resize: vertical;
		transition: border-color 0.2s;
		box-sizing: border-box;
	}

	textarea:focus {
		outline: none;
		border-color: #4a90e2;
		background-color: #fff;
	}
</style>
