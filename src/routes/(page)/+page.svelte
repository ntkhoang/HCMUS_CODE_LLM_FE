<script lang="ts">
    import { Button } from "$lib/components/ui/button";
    import { Input } from "$lib/components/ui/input";
    import { Card, CardContent } from "$lib/components/ui/card";
    import { ScrollArea } from "$lib/components/ui/scroll-area";
    import { Alert, AlertDescription } from "$lib/components/ui/alert";
    import { CircleAlert, Send, User, Pencil, Check, X, BotIcon } from "lucide-svelte";
    import { Toaster, toast } from "svelte-sonner";
    import { Trash2 } from "lucide-svelte";
    import { Moon, Sun, Settings2 } from "lucide-svelte";

    let textareaElement: HTMLTextAreaElement;
    
    function resetTextarea() {
        if (textareaElement) {
            textareaElement.style.height = 'auto';
            inputText = '';
        }
    }

    let isDark = $state(false);
    
    function toggleTheme() {
        isDark = !isDark;
        document.documentElement.classList.toggle('dark');
    }

    let inputText = $state("");
    let editText = $state("");
    let editingIndex = $state<number | null>(null);

	let url = "https://6c9f-34-171-147-5.ngrok-free.app/ask"

    type Message = {
        type: 'user' | 'ai' | 'error';
        content: string;
    };

    let messages = $state<Message[]>([]);
    let isLoading = $state(false);
    let context: string = "";

    function resetChat() {
        if (messages.length === 0) {
            toast.error("Chat is already empty");
            return;
        }
        messages = [];
        context = "";
        toast.success("Chat history cleared");
    }

    function startEditing(index: number) {
        editingIndex = index;
        editText = messages[index].content;
    }

    function cancelEditing() {
        editingIndex = null;
        editText = "";
    }

    function generateContext() {
		let newContext = '';
		for (let i = 0; i < messages.length; i++) {
			if (messages[i].type === 'user') {
				newContext += 'Instruction: ' + messages[i].content + '\nResponse: ';
			} else if (messages[i].type === 'ai') {
				newContext += messages[i].content + '\n';
			}
		}
		return newContext;
	}

	async function saveEdit(index: number) {
		if (!editText.trim()) {
			toast.error("Message cannot be empty");
			return;
		}

		isLoading = true;
		messages = [
			...messages.slice(0, index),
			{ type: 'user', content: editText }
		];
		
		context = generateContext();
		
		try {
			const response = await fetch(url, {
				method: "POST",
				headers: {
					"Content-Type": "application/json",
				},
				body: JSON.stringify({ question: context }),
			});

			if (response.ok) {
				const data = await response.json();
				messages = [...messages, { type: 'ai', content: data.answer }];
				context = generateContext();
			} else {
				throw new Error(response.statusText);
			}
		} catch (error) {
			toast.error("Failed to update response");
			messages = [...messages, { type: 'error', content: "Failed to update response" }];
		} finally {
			isLoading = false;
			editingIndex = null;
			editText = "";
		}
	}

    async function sendMessage() {
        if (!inputText.trim()) {
            toast.error("Please enter a message");
            return;
        }
        isLoading = true;
        context += 'Instruction: ' + inputText + '\nResponse: ';
        messages = [...messages, { type: 'user', content: inputText }];

        try {
            const response = await fetch(url, {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({ question: context }),
            });
            inputText = "";
            
            if (response.ok) {
                const data = await response.json();
                messages = [...messages, { type: 'ai', content: data.answer }];
                context += data.answer + '\n';
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


<div class="flex flex-col h-screen w-full max-w-6xl mx-auto p-6">
    <div class="relative">
        <Button 
            variant="ghost" 
            size="icon"
            on:click={toggleTheme}
            class="rounded-full hover:bg-accent"
        >
            {#if isDark}
                <Moon class="h-5 w-5" />
            {:else}
                <Sun class="h-5 w-5" />
            {/if}
        </Button>
    </div>
    <div class="flex flex-col h-screen w-full max-w-6xl mx-auto p-6 ">
        <h1 class="text-2xl font-bold mb-4">AI Chat Interface</h1>
        <Card class="flex-grow mb-4">
            <ScrollArea class="h-[calc(100vh-220px)]">
                <CardContent class="p-6 space-y-6">
                    {#each messages as message, index}
                        <div class="mb-4 {message.type === 'user' ? 'text-right' : 'text-left'}">
                            <div class="inline-block max-w-[75%] p-4 pt-8 rounded-lg whitespace-pre-wrap relative group
                                {message.type === 'user' ? 'bg-primary text-primary-foreground' : 
                                message.type === 'ai' ? 'bg-secondary' : 'bg-destructive text-destructive-foreground'}">
                                
                                {#if message.type === 'user'}
                                    <div class="absolute -top-2 -left-2 bg-primary rounded-full p-1 shadow-md">
                                        <User class="h-5 w-5" />
                                    </div>
                                    {#if editingIndex === index}
                                        <div class="flex items-center gap-2">
                                            <Input
                                                type="text"
                                                bind:value={editText}
                                                class="min-w-[200px] bg-primary text-primary-foreground placeholder:text-primary-foreground/50"
                                                autofocus
                                            />
                                            <Button 
                                                size="icon" 
                                                variant="secondary"
                                                on:click={() => saveEdit(index)}
                                                disabled={isLoading}
                                                class="bg-primary text-primary-foreground"
                                            >
                                                <Check class="h-4 w-4" />
                                            </Button>
                                            <Button 
                                                size="icon" 
                                                variant="secondary"
                                                on:click={cancelEditing}
                                                class="bg-primary text-primary-foreground"
                                            >
                                                <X class="h-4 w-4" />
                                            </Button>
                                        </div>
                                    {:else}
                                        <span>{message.content}</span>
                                        <Button
                                            size="icon"
                                            variant="ghost"
                                            class="opacity-0 group-hover:opacity-100 absolute -left-8 top-1/2 transform -translate-y-1/2 text-primary-foreground"
                                            on:click={() => startEditing(index)}
                                        >
                                            <Pencil class="h-4 w-4" />
                                        </Button>
                                    {/if}
                                {:else}
                                    <div class="absolute -top-2 -left-2 bg-secondary rounded-full p-1 shadow-md">
                                        <BotIcon class="h-5 w-5" />
                                    </div>
                                    {message.content}
                                {/if}
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
            <Button 
                variant="destructive"
                size="sm"
                on:click={resetChat}
                disabled={isLoading}
                class="hover:bg-destructive/90"
            >
                <Trash2 class="mr-2 h-4 w-4" />
                Clear Chat
            </Button>
            <textarea
                bind:this={textareaElement}
                bind:value={inputText}
                placeholder="Type your message here..."
                disabled={isLoading}
                rows="1"
                class="flex-grow resize-none overflow-y-auto rounded-md border border-input 
                    bg-background px-4 py-3 text-sm ring-offset-background 
                    placeholder:text-muted-foreground focus-visible:outline-none 
                    focus-visible:ring-2 focus-visible:ring-ring focus-visible:ring-offset-2 
                    disabled:cursor-not-allowed disabled:opacity-50 min-h-[40px] max-h-[200px]"
                oninput={(e) => {
                    const target = e.target as HTMLTextAreaElement;
                    target.style.height = 'auto';
                    target.style.height = Math.min(target.scrollHeight, 75) + 'px';
                    target.scrollTop = target.scrollHeight;
                }}
                onkeydown={(e) => {
                    if (e.key === 'Enter' && !e.shiftKey) {
                        e.preventDefault();
                        sendMessage();
                        resetTextarea();
                    }
                }}
            ></textarea>
            <Button type="submit" disabled={isLoading || !inputText}>
                <Send class="mr-2 h-4 w-4" />
                Send
            </Button>
        </form>
    </div>

    <Toaster />
</div>