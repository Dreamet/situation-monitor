<script lang="ts">
	import type { NewsItem } from '$lib/types';
	import { onMount } from 'svelte';

	interface Props {
		item: NewsItem;
	}

	let { item }: Props = $props();
	
	let translatedHeadline = $state(''); // 用來存中文翻譯

	// 當這個新聞方塊出現時，執行這段翻譯程式
	onMount(async () => {
		try {
			// 這是 Google 翻譯的免費小門，雖然不保證永遠有效，但個人用很足夠
			const url = `https://translate.googleapis.com/translate_a/single?client=gtx&sl=auto&tl=zh-TW&dt=t&q=${encodeURIComponent(item.headline)}`;
			const res = await fetch(url);
			const data = await res.json();
			// 抓出翻譯後的文字
			if (data && data[0] && data[0][0] && data[0][0][0]) {
				translatedHeadline = data[0][0][0];
			}
		} catch (e) {
			console.error('翻譯失敗', e);
			// 失敗就算了，維持空白，顯示原文就好
		}
	});
</script>

<a 
	href={`https://translate.google.com/translate?sl=auto&tl=zh-TW&u=${encodeURIComponent(item.url)}`}
	target="_blank" 
	rel="noopener noreferrer" 
	class="news-item"
>
	<div class="content">
		<div class="header">
			<span class="source">{item.source}</span>
			<span class="time">{new Date(item.datetime).toLocaleTimeString()}</span>
		</div>
		
		<h3 class="headline cn">
			{translatedHeadline || '翻譯中...'}
		</h3>
		
		<div class="headline en">
			{item.headline}
		</div>

		{#if item.summary}
			<p class="summary">{item.summary}</p>
		{/if}
	</div>
</a>

<style>
	.news-item {
		display: block;
		padding: 0.75rem;
		text-decoration: none;
		color: inherit;
		border-bottom: 1px solid var(--border-color);
		transition: background-color 0.2s;
	}

	.news-item:hover {
		background-color: var(--bg-hover);
	}

	.news-item:last-child {
		border-bottom: none;
	}

	.header {
		display: flex;
		justify-content: space-between;
		font-size: 0.7rem;
		color: var(--text-secondary);
		margin-bottom: 0.25rem;
	}

	.source {
		font-weight: 600;
		color: var(--accent-color);
	}

	/* 中文標題樣式 */
	.headline.cn {
		margin: 0 0 0.25rem 0;
		font-size: 0.95rem;
		line-height: 1.4;
		font-weight: 500;
		color: var(--text-primary);
	}

	/* 英文原文樣式 */
	.headline.en {
		margin: 0 0 0.5rem 0;
		font-size: 0.75rem;
		line-height: 1.3;
		color: var(--text-secondary);
		opacity: 0.8;
	}

	.summary {
		margin: 0;
		font-size: 0.8rem;
		color: var(--text-secondary);
		display: -webkit-box;
		-webkit-line-clamp: 2;
		-webkit-box-orient: vertical;
		overflow: hidden;
	}
</style>
