<script lang="ts">
	import type { NewsItem } from '$lib/types';
	import { timeAgo } from '$lib/utils';
	import { onMount } from 'svelte'; // 新增這個

	interface Props {
		item: NewsItem;
		showSource?: boolean;
		showAlert?: boolean;
		showDescription?: boolean;
		compact?: boolean;
	}

	let {
		item,
		showSource = true,
		showAlert = true,
		showDescription = false,
		compact = false
	}: Props = $props();

	// 新增：用來存翻譯後的中文
	let translatedTitle = $state('');

	// 新增：組件載入時，偷偷去問 Google 翻譯
	onMount(async () => {
		if (!item.title) return;
		try {
			// 使用 Google Translate 免費接口
			const url = `https://translate.googleapis.com/translate_a/single?client=gtx&sl=auto&tl=zh-TW&dt=t&q=${encodeURIComponent(item.title)}`;
			const res = await fetch(url);
			const data = await res.json();
			// 取得翻譯結果
			if (data && data[0] && data[0][0] && data[0][0][0]) {
				translatedTitle = data[0][0][0];
			}
		} catch (e) {
			console.error('Translation failed', e);
		}
	});

	// 新增：產生 Google 翻譯網頁的連結
	const translateLink = $derived(
		`https://translate.google.com/translate?sl=auto&tl=zh-TW&u=${encodeURIComponent(item.link)}`
	);
</script>

<div class="news-item" class:alert={showAlert && item.isAlert} class:compact>
	{#if showSource}
		<div class="item-source">
			{item.source}
			{#if showAlert && item.isAlert}
				<span class="alert-tag">ALERT</span>
			{/if}
		</div>
	{/if}

	<a class="item-title-wrapper" href={translateLink} target="_blank" rel="noopener noreferrer">
		<span class="title-cn">
			{translatedTitle || '...'}
		</span>
		<span class="title-en">
			{item.title}
		</span>
	</a>

	{#if showDescription && item.description}
		<p class="item-description">{item.description}</p>
	{/if}

	<div class="item-meta">
		<span class="item-time">{timeAgo(item.timestamp)}</span>
		{#if item.region}
			<span class="item-region">{item.region}</span>
		{/if}
	</div>
</div>

<style>
	.news-item {
		padding: 0.5rem 0;
		border-bottom: 1px solid var(--border);
	}

	.news-item:last-child {
		border-bottom: none;
	}

	.news-item.compact {
		padding: 0.35rem 0;
	}

	.news-item.alert {
		background: rgba(255, 68, 68, 0.08);
		margin: 0 -0.5rem;
		padding: 0.5rem;
		border-radius: 4px;
		border: 1px solid rgba(255, 68, 68, 0.2);
		border-bottom: 1px solid rgba(255, 68, 68, 0.2);
	}

	.item-source {
		font-size: 0.55rem;
		color: var(--text-secondary);
		text-transform: uppercase;
		letter-spacing: 0.03em;
		margin-bottom: 0.2rem;
		display: flex;
		align-items: center;
		gap: 0.4rem;
	}

	.alert-tag {
		background: var(--danger);
		color: white;
		font-size: 0.5rem;
		padding: 0.1rem 0.3rem;
		border-radius: 2px;
		font-weight: 600;
	}

	/* 修改：標題外層容器 */
	.item-title-wrapper {
		display: flex;
		flex-direction: column; /* 讓中文英文上下排列 */
		text-decoration: none;
		gap: 0.1rem;
	}

	.item-title-wrapper:hover .title-cn {
		color: var(--accent);
	}

	/* 新增：中文標題樣式 */
	.title-cn {
		font-size: 0.85rem; /* 比原本大一點，方便閱讀中文 */
		line-height: 1.4;
		color: var(--text-primary);
		font-weight: 500;
	}

	/* 新增：英文標題樣式 */
	.title-en {
		font-size: 0.6rem;
		line-height: 1.2;
		color: var(--text-muted); /* 顏色淡一點 */
		opacity: 0.8;
	}

	.compact .title-cn {
		font-size: 0.75rem;
	}

	.item-description {
		font-size: 0.6rem;
		color: var(--text-secondary);
		margin: 0.3rem 0 0;
		line-height: 1.4;
	}

	.item-meta {
		display: flex;
		align-items: center;
		gap: 0.5rem;
		margin-top: 0.25rem;
	}

	.item-time {
		font-size: 0.55rem;
		color: var(--text-muted);
	}

	.item-region {
		font-size: 0.5rem;
		color: var(--accent);
		background: rgba(var(--accent-rgb), 0.1);
		padding: 0.1rem 0.3rem;
		border-radius: 2px;
		text-transform: uppercase;
	}
</style>
