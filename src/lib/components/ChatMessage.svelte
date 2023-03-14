<script lang="ts">
	import type { ChatCompletionRequestMessageRoleEnum } from 'openai'
	export let type: ChatCompletionRequestMessageRoleEnum
	export let message: string
</script>

<div class="chat {type === 'user' ? 'chat-end' : 'chat-start'} justify-end">
	<div class="chat-image avatar">
		<div class="w-10 rounded-full">
			<img
				src="https://ui-avatars.com/api/?name={type === 'user' ? 'Me' : 'B'}"
				alt="{type} avatar"
			/>
		</div>
	</div>
	<div class="chat-header">
		{type === 'user' ? 'Me' : 'GPT3.5'}
	</div>
	<div class="chat-bubble whitespace-pre-wrap {type === 'user' ? 'chat-bubble-primary' : 'chat-bubble-info bg-gray-700 bg-gradient-to-r from-gray-700 to-gray-600 text-gray-200'}">
		{message}
	</div>
</div>
