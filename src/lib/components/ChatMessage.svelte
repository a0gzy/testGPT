<script lang="ts">
	import type { ChatCompletionRequestMessageRoleEnum, ChatCompletionRequestMessage } from 'openai'
	export let type: ChatCompletionRequestMessageRoleEnum
	export let message: string

	export let chatMessages: ChatCompletionRequestMessage[] = [];

	const handleDel = async () => {
		chatMessages = chatMessages.filter((e) => e.content !== message)
		console.log(chatMessages)

		const stringChatMessages = JSON.stringify(chatMessages)
		localStorage.setItem("chatMessages",stringChatMessages)
	}

	let hover: Boolean;
</script>

<div class="chat {type === 'user' ? 'chat-end' : 'chat-start'} justify-end">
	<div class="chat-image avatar cursor-pointer">
		<div class="w-10 rounded-full bg-secondary-content">
			<button class="w-full h-full flex justify-center items-center" 
				on:mouseenter={() => {hover = true}} on:mouseleave={() => {hover = false}} 
				on:click={handleDel}>
				{#if hover}
					<h1 class="text-neutral-focus text-center text-xl">✖</h1>
				{:else}
					<h1 class="text-neutral-focus text-center text-lg">{type === 'user' ? 'Me' : 'B'}</h1>
				{/if}
			</button>
			
			<!-- <img
				src="https://ui-avatars.com/api/?name={type === 'user' ? 'Me' : 'B'}"
				alt="{type} avatar"
				
			/> -->
		</div>
	</div>
	<div class="chat-header">
		{type === 'user' ? 'Me' : 'GPT3.5'}
	</div>
	<div class="chat-bubble whitespace-pre-wrap {type === 'user' ? 'chat-bubble-primary' : 'chat-bubble-info bg-gray-700 bg-gradient-to-r from-gray-700 to-gray-600 text-gray-200'}">
		{message}
	</div>
</div>
