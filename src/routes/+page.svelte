<script lang="ts">
	import ChatMessage from '$lib/components/ChatMessage.svelte'
	import type { ChatCompletionRequestMessage } from 'openai'
	import { SSE } from 'sse.js'
	import { getTokens } from '$lib/tokenizer'

	let query: string = ''
	let answer: string = ''
	let loading: boolean = false
	let chatMessages: ChatCompletionRequestMessage[] = []
	let scrollToDiv: HTMLDivElement

	function scrollToBottom() {
		setTimeout(function () {
			scrollToDiv.scrollIntoView({ behavior: 'smooth', block: 'end', inline: 'nearest' })
		}, 100)
	}

	function getAllTokens(){
		let tokenCount = 0

		chatMessages.forEach((msg) => {
			const tokens = getTokens(msg.content)
			tokenCount += tokens
		})

		const prompt =
			'Ты виртуальный ассистент, который может ответить почти на каждый вопрос.'
		tokenCount += getTokens(prompt)

		// if (tokenCount >= 4000) {
		// 	chatMessages = []
		// }

		return tokenCount
	}

	const handleSubmit = async () => {
		loading = true

		if(chatMessages.length > 1){
			const lastMsg = chatMessages[chatMessages.length - 1]
			if(lastMsg.content === "Следующий вопрос обновит запросы"){
				chatMessages = []
			}
		}

		chatMessages = [...chatMessages, { role: 'user', content: query }]

		const eventSource = new SSE('/api/chat', {
			headers: {
				'Content-Type': 'application/json'
			},
			payload: JSON.stringify({ messages: chatMessages })
		})

		query = ''

		eventSource.addEventListener('error', handleError)

		eventSource.addEventListener('message', (e) => {

			scrollToBottom()
			try {
				loading = false
				if (e.data === '[DONE]') {
					chatMessages = [...chatMessages, { role: 'assistant', content: answer }]
					const tokenCounts = getAllTokens()
					if(tokenCounts >= 4000){
						chatMessages = [...chatMessages, { role: 'assistant', content: "Следующий вопрос обновит запросы" }]
					}
					answer = ''
					return
				}

				const completionResponse = JSON.parse(e.data)
				const [{ delta }] = completionResponse.choices

				if (delta.content) {
					answer = (answer ?? '') + delta.content
				}
			} catch (err) {
				if(err instanceof Error){
					if(err.name == "Query too large"){
						console.log("qtl")
						chatMessages = []
					}
				}
				handleError(err)
			}
		})
		eventSource.stream()
		scrollToBottom()
	}

	function handleError<T>(err: T) {
		loading = false
		query = ''
		answer = ''
		console.error(err)
		if(err instanceof Error){
			if(err.name == "Query flagged by openai"){
				chatMessages = [...chatMessages, { role: 'assistant', content: "OpenAi moderation error" }]
			}
		}
		if (getAllTokens() >= 4000) {
			chatMessages = []
		}
	}
</script>

<div class="flex flex-col pt-4 w-full h-screen px-8 items-center gap-2">
	<div class="h-[calc(100%-100px)] w-full bg-gray-900 rounded-md p-4 overflow-y-auto flex flex-col gap-4">
		<div class="flex flex-col gap-2">
			<ChatMessage type="assistant" message="Привет, спрашивайте меня что хотите!" />
			{#each chatMessages as message}
				<ChatMessage type={message.role} message={message.content} />
			{/each}
			{#if answer}
				<ChatMessage type="assistant" message={answer} />
			{/if}
			{#if loading}
				<ChatMessage type="assistant" message="Loading.." />
			{/if}
		</div>
		<div class="" bind:this={scrollToDiv} />
	</div>
	<form
		class="flex w-full rounded-md gap-4 bg-gray-900 p-4"
		on:submit|preventDefault={() => handleSubmit()}
	>
		<input type="text" class="input input-bordered w-full dark:text-white text-black" bind:value={query} />
		<button type="submit" class="btn btn-accent"> Send </button>
	</form>
</div>
