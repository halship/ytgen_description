<script lang="ts">
	import type { Tag, UsedTool, UsedMaterial } from '$lib/types';

	let tags = $state<Tag[]>([]);
	let summary = $state('');
	let copied = $state(false);
	let playedGame = $state('');
	let usedTools = $state<UsedTool[]>([]);
	let usedMaterials = $state<UsedMaterial[]>([]);
	let output = $derived.by(() => {
		const tagsResult = tags.map((tag) => `#${tag.name}`).join(' ');
		const playedGameResult = playedGame.trim() !== '' ? `【プレイするゲーム】\n${playedGame}` : '';
		const usedToolsResult =
			usedTools.length > 0
				? `【使用したツール】\n${usedTools.map((tool) => `・${tool.name}\n  ${tool.url}`).join('\n')}`
				: '';
		const usedMaterialsResult =
			usedMaterials.length > 0
				? `【使用した素材】\n${usedMaterials.map((material) => `・${material.name}\n  ${material.url}`).join('\n')}`
				: '';

		return [tagsResult, summary, playedGameResult, usedToolsResult, usedMaterialsResult]
			.filter((result) => result.trim() !== '')
			.join('\n\n')
			.trim();
	});

	function addTag() {
		tags = [
			...tags,
			{ id: tags.length > 0 ? Math.max(...tags.map((tag) => tag.id)) + 1 : 0, name: '' }
		];
	}

	function removeTag(id: number) {
		tags = tags.filter((tag) => tag.id !== id);
	}

	function addUsedTool() {
		usedTools = [
			...usedTools,
			{
				id: usedTools.length > 0 ? Math.max(...usedTools.map((tool) => tool.id)) + 1 : 0,
				name: '',
				url: ''
			}
		];
	}

	function removeUsedTool(id: number) {
		usedTools = usedTools.filter((tool) => tool.id !== id);
	}

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

	function addUsedMaterial() {
		usedMaterials = [...usedMaterials, { id: usedMaterials.length, name: '', url: '' }];
	}

	function removeUsedMaterial(id: number) {
		usedMaterials = usedMaterials.filter((material) => material.id !== id);
	}

	function exportToJson() {
		const json = {
			tags: tags.map((tag) => tag.name),
			summary,
			playedGame: playedGame.trim() !== '' ? playedGame : null,
			usedTools: usedTools.map((tool) => ({ name: tool.name, url: tool.url })),
			usedMaterials: usedMaterials.map((material) => ({ name: material.name, url: material.url }))
		};
		const jsonString = JSON.stringify(json, null, 2);
		const blob = new Blob([jsonString], { type: 'application/json' });
		const url = URL.createObjectURL(blob);
		const a = document.createElement('a');
		a.href = url;
		a.download = 'youtube_description.json';
		a.click();
	}

	function importFromJson(e: Event) {
		const target = e.target as HTMLInputElement;
		const file = target.files?.[0];
		if (!file) return;
		const reader = new FileReader();
		reader.onload = (e) => {
			const jsonString = e.target?.result as string;
			const json = JSON.parse(jsonString);
			tags = json.tags.map((tag: string, index: number) => ({ id: index, name: tag }));
			summary = json.summary;
			playedGame = json.playedGame;
			usedTools = json.usedTools.map((tool: { name: string; url: string }, index: number) => ({
				id: index,
				name: tool.name,
				url: tool.url
			}));
			usedMaterials = json.usedMaterials.map(
				(material: { name: string; url: string }, index: number) => ({
					id: index,
					name: material.name,
					url: material.url
				})
			);
		};
		reader.readAsText(file);
	}
</script>

<div class="container">
	<h1>Youtube説明文ジェネレータ</h1>

	<section class="section">
		<h2>タグ入力欄</h2>
		<div class="tag-input-container">
			{#each tags as tag (tag.id)}
				<div class="tag-row">
					<input type="text" class="tag-input" bind:value={tag.name} placeholder="タグ名を入力" />
					<button class="btn btn-danger" onclick={() => removeTag(tag.id)}>削除</button>
				</div>
			{/each}
			<button class="btn btn-primary" onclick={addTag}>タグ追加</button>
		</div>
	</section>

	<section class="section">
		<h2>概要入力欄</h2>
		<textarea
			class="summary-textarea"
			rows={10}
			cols={80}
			bind:value={summary}
			placeholder="概要を入力"
		></textarea>
	</section>

	<section class="section">
		<h2>プレイするゲーム入力欄</h2>
		<input
			type="text"
			class="played-game-input"
			bind:value={playedGame}
			placeholder="プレイしたゲームを入力"
		/>
	</section>

	<section class="section">
		<h2>使用したツール入力欄</h2>
		<div class="used-tool-input-container">
			{#each usedTools as usedTool (usedTool.id)}
				<div class="used-tool-row">
					<input
						type="text"
						class="used-tool-input"
						bind:value={usedTool.name}
						placeholder="使用したツールを入力"
					/>
					<input
						type="text"
						class="used-tool-url-input"
						bind:value={usedTool.url}
						placeholder="使用したツールのURLを入力"
					/>
					<button class="btn btn-danger" onclick={() => removeUsedTool(usedTool.id)}>削除</button>
				</div>
			{/each}
			<button class="btn btn-primary" onclick={addUsedTool}>使用したツール追加</button>
		</div>
	</section>

	<section class="section">
		<h2>使用した素材入力欄</h2>
		<div class="used-material-input-container">
			{#each usedMaterials as usedMaterial (usedMaterial.id)}
				<div class="used-material-row">
					<input
						type="text"
						class="used-material-input"
						bind:value={usedMaterial.name}
						placeholder="使用した素材を入力"
					/>
					<input
						type="text"
						class="used-material-url-input"
						bind:value={usedMaterial.url}
						placeholder="使用した素材のURLを入力"
					/>
					<button class="btn btn-danger" onclick={() => removeUsedMaterial(usedMaterial.id)}
						>削除</button
					>
				</div>
			{/each}
			<button class="btn btn-primary" onclick={addUsedMaterial}>使用した素材追加</button>
		</div>
	</section>

	<hr />

	<section class="section">
		<h2>出力結果</h2>
		<div class="output-container">
			<textarea class="output-textarea" rows={10} cols={80} readonly>{output}</textarea>
			<div class="output-footer">
				<button class="btn btn-copy" onclick={copyToClipboard}>
					{copied ? 'コピーしました！' : '一括コピー'}
				</button>
			</div>
		</div>
	</section>

	<hr />

	<section class="section">
		<div class="import-export-container">
			<button class="btn btn-primary" onclick={exportToJson}>エクスポート</button>
			<label for="file-input" class="btn btn-secondary">インポート</label>
			<input
				id="file-input"
				type="file"
				class="file-input"
				onchange={importFromJson}
				accept="application/json"
			/>
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

	hr {
		margin: 2rem 0;
		border: 0;
		border-top: 1px solid #ddd;
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

	.tag-input {
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

	.tag-input:focus {
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

	.summary-textarea {
		width: 100%;
		padding: 1rem;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 1rem;
		resize: vertical;
		transition: border-color 0.2s;
		box-sizing: border-box;
	}

	.summary-textarea:focus {
		outline: none;
		border-color: #4a90e2;
		background-color: #fff;
		box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
	}

	.played-game-input {
		width: 100%;
		padding: 0.75rem;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 1rem;
		transition: border-color 0.2s;
		box-sizing: border-box;
	}

	.played-game-input:focus {
		outline: none;
		border-color: #4a90e2;
		box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
	}

	.output-textarea {
		width: 100%;
		padding: 1rem;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 1rem;
		resize: vertical;
		transition: border-color 0.2s;
		box-sizing: border-box;
		background-color: #f9f9f9;
	}

	.used-tool-input-container {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
	}

	.used-tool-row {
		display: flex;
		flex-direction: row;
		gap: 0.75rem;
		align-items: center;
	}

	.used-tool-input {
		flex: 1;
		padding: 0.75rem;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 1rem;
		transition: border-color 0.2s;
		box-sizing: border-box;
	}

	.used-tool-input:focus {
		outline: none;
		border-color: #4a90e2;
		box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
	}

	.used-tool-url-input {
		flex: 1;
		padding: 0.75rem;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 1rem;
		transition: border-color 0.2s;
		box-sizing: border-box;
	}

	.used-tool-url-input:focus {
		outline: none;
		border-color: #4a90e2;
		box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
	}

	.used-material-input-container {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
	}

	.used-material-row {
		display: flex;
		flex-direction: row;
		gap: 0.75rem;
		align-items: center;
	}

	.used-material-input {
		flex: 1;
		padding: 0.75rem;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 1rem;
		transition: border-color 0.2s;
		box-sizing: border-box;
	}

	.used-material-input:focus {
		outline: none;
		border-color: #4a90e2;
		box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
	}

	.used-material-url-input {
		flex: 1;
		padding: 0.75rem;
		border: 1px solid #ddd;
		border-radius: 4px;
		font-size: 1rem;
		transition: border-color 0.2s;
		box-sizing: border-box;
	}

	.used-material-url-input:focus {
		outline: none;
		border-color: #4a90e2;
		box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
	}

	.import-export-container {
		display: flex;
		gap: 0.75rem;
		align-items: center;
	}

	.file-input {
		position: absolute;
		width: 0;
		height: 0;
		opacity: 0;
		overflow: hidden;
	}

	.btn-secondary {
		background-color: #95a5a6;
		color: white;
	}

	.btn-secondary:hover {
		background-color: #7f8c8d;
	}
</style>
