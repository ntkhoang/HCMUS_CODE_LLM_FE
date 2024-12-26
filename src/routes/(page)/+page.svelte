<script lang="ts">
	import { Button } from "$lib/components/ui/button";
	import { Input } from "$lib/components/ui/input";
	import { Card, CardContent } from "$lib/components/ui/card";
	import { ScrollArea } from "$lib/components/ui/scroll-area";
	import { Alert, AlertDescription } from "$lib/components/ui/alert";
	import { CircleAlert, Send, User } from "lucide-svelte";;
	import { Toaster, toast } from "svelte-sonner";

	let inputText = $state("");
	type Message = {
		type: 'user' | 'ai' | 'error';
		content: string;
	};


	let messages = $state<Message[]>([]);
	let isLoading = $state(false);
	let context : string = "";
	async function sendMessage() {
		if (!inputText.trim()) {
			toast.error("Please enter a message");
			return;
		}
		isLoading = true;
		context+= 'Instruction: '+ inputText + '\nResponse: ' ;
		messages = [...messages, { type: 'user', content: inputText }];

		const url = "https://ae44-34-105-117-107.ngrok-free.app/ask";
		try {
			const response = await fetch(url, {
				method: "POST",
				headers: {
					"Content-Type": "application/json",
				},
				body: JSON.stringify({ question: context}),
			});

			if (response.ok) {
				const data = await response.json();
				messages = [...messages, { type: 'ai', content: data.answer }];
				context+= data.answer + '\n';
			} else {
				throw new Error(response.statusText);
			}
		} catch (error) {
			console.error(error);
			toast.error("Failed to get response");
			messages = [...messages, { type: 'error', content: "Failed to get response. Please try again." }];
		} finally {
			isLoading = false;
		}
	}
	function preventDefault(fn: (e: Event) => void) {
		return (e: Event) => {
			e.preventDefault();
			fn(e);
		};
	}
</script>

<div class="flex flex-col h-screen max-w-3xl mx-auto p-4">
	<h1 class="text-2xl font-bold mb-4">AI Chat Interface</h1>

	<Card class="flex-grow mb-4">
		<ScrollArea class="h-[calc(100vh-200px)]">
			<CardContent>
				{#each messages as message}
					<div class="mb-4 {message.type === 'user' ? 'text-right' : 'text-left'}">
						<div class="inline-block max-w-[70%] p-3 rounded-lg whitespace-pre-wrap {message.type === 'user' ? 'bg-primary text-primary-foreground' : message.type === 'ai' ? 'bg-secondary' : 'bg-destructive text-destructive-foreground'}">
							{#if message.type === 'user'}
								<User class="inline-block mr-2 h-4 w-4" />
							{:else if message.type === 'ai'}
								<CircleAlert class="inline-block mr-2 h-4 w-4" />
							{/if}
							{message.content}
						</div>
					</div>
				{/each}
				{#if isLoading}
					<div class="text-center">
						<CircleAlert class="inline-block h-6 w-6 animate-spin" />
						<p class="mt-2">AI is thinking...</p>
					</div>
				{/if}
			</CardContent>
		</ScrollArea>
	</Card>

	<form onsubmit={preventDefault(() => sendMessage())} class="flex gap-2">
		<Input 
			type="text" 
			bind:value={inputText} 
			placeholder="Type your message here..." 
			disabled={isLoading}
			class="flex-grow"
		/>
		<Button type="submit" disabled={isLoading || !inputText.trim()}>
			<Send class="mr-2 h-4 w-4" />
			Send
		</Button>
	</form>
</div>
<Toaster />