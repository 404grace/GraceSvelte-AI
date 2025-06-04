<script lang="ts">
	export let data;

	let chats = data.chats;
	let sidebarOpen = true;

	let messages = [];
	let currentChatId: number | null = null;

	let newMessage = '';
	let isLoading = false;

	function switchChat(newChatId: number) {
		const chat = chats.find((c) => c.id === newChatId);
		messages = chat.messages;
		currentChatId = newChatId;
	}

	async function handleSubmit() {
		isLoading = true;
		messages = [...messages, { content: newMessage, userType: 'human' }];
		const messageContent = newMessage;
		newMessage = '';

		const response = await fetch('/api/chat', {
			method: 'POST',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify({
				message: messageContent,
				chatId: currentChatId
			})
		});

		const data = await response.json();

		messages = [...messages, { content: data.response, userType: 'assistant' }];

		if (currentChatId !== data.chatId) {
			chats = [{ id: data.chatId, title: 'New Chat', messages: messages }, ...chats];
			currentChatId = data.chatId;
		} else {
			chats = chats.map((c) => {
				if (c.id === data.chatId) {
					return {
						...c,
						messages: messages
					};
				}
				return c;
			});
		}

		isLoading = false;
	}
</script>

<main class="flex h-screen">
	<!-- Sidebar -->
	<div class="w-64 bg-gray-200 flex flex-col {sidebarOpen ? '' : 'hidden'}">
		{#each chats as c}
			<button
				on:click={() => switchChat(c.id)}
				class="w-full p-2 rounded-sm {currentChatId == c.id ? 'bg-slate-300' : 'bg-slate-200'}"
				>{c.title}</button
			>
		{/each}
	</div>

	<!-- Messages -->
	<div class="flex-1 flex flex-col">
		<div class="flex justify-start bg-gray-300 border-b">
			<button on:click={() => (sidebarOpen = !sidebarOpen)} class="p-2">x</button>
		</div>
		<div class="flex-1 overflow-y-auto space-y-4">
			{#each messages as m}
				<div class="p-4 flex {m.userType == 'human' ? 'justify-end' : 'justify-start'}">
					<div
						class="px-4 py-2 max-w-[80%] rounded-md {m.userType == 'human'
							? 'bg-blue-500'
							: 'bg-gray-200'}"
					>
						{m.content}
					</div>
				</div>
			{/each}
		</div>

		<form class="bg-slate-200 p-4 flex" on:submit={handleSubmit}>
			<input
				class="flex-1 border border-black rounded-md"
				bind:value={newMessage}
				placeholder="Ask anything ...."
			/>
			<button class="px-4 py-2 bg-blue-600 text-white rounded-md" type="submit" disabled={isLoading}
				>Submit</button
			>
		</form>
	</div>
</main>
