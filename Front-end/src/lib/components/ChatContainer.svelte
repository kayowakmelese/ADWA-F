<script>
    import { marked } from "marked";
    // Import the environment variable for the API URL
    import { public_env } from '$env/static/public'; 

    let messages = $state([
        {
            role: "assistant",
            content: "Selam! I am Adwa AI. Ask me anything about the Battle of Adwa and the people behind Ethiopia’s historic victory.",
        },
    ]);
    let input = $state("");
    let isTyping = $state(false);

    const API_BASE_URL = PUBLIC_API_URL || "http://localhost:8000";

    $effect(() => {
        messages.length;
        const container = document.querySelector(".custom-scrollbar");
        if (container) {
            queueMicrotask(() => {
                container.scrollTo({
                    top: container.scrollHeight,
                    behavior: "smooth",
                });
            });
        }
    });

    async function sendMessage() {
        if (!input.trim() || isTyping) return;

        const question = input;
        messages.push({ role: "user", content: question });
        input = "";
        isTyping = true;

        try {
            // FIXED: Using the dynamic API_BASE_URL
            const url = `${API_BASE_URL}/response/?question=${encodeURIComponent(question)}`;
            const res = await fetch(url);

            if (!res.ok) throw new Error("Failed to connect to backend");

            const data = await res.json();

            // Your robust extraction logic remains the same...
            const replyText =
                (data?.answer?.answer) ||
                (typeof data?.answer === "string" && data.answer) ||
                (typeof data === "string" && data) ||
                "I couldn't process that response.";

            const sources = data?.source?.source || data?.sources || data?.source || [];

            messages.push({
                role: "assistant",
                content: replyText,
                sources,
                showSources: false,
            });
        } catch (error) {
            console.error("Fetch error:", error);
            messages.push({
                role: "assistant",
                content: "⚠️ Connection error. Please try again in a moment.",
            });
        } finally {
            isTyping = false;
        }
    }
</script>












<!-- Main Container -->
<div
    class="flex flex-col
    w-full h-[calc(100dvh-80px)]
    sm:h-[80vh] sm:max-h-[850px] sm:max-w-4xl sm:mx-auto sm:mt-10 sm:rounded-3xl
    border border-white/10 bg-black/5 backdrop-blur-2xl shadow-2xl overflow-hidden text-lg sm:text-xl"
>
    <!-- 1. Message Display Area -->
    <div class="flex-1 overflow-y-auto p-6 space-y-4 custom-scrollbar">
        {#each messages as msg}
            {#if msg.role === "assistant"}
                <div class="flex justify-start">
                    <div
                        class="max-w-[80%] rounded-2xl rounded-tl-none bg-black/20 p-4 text-white border border-white/5"
                    >
                        <p class="text-sm font-bold text-orange-400 mb-1">
                            Adwa AI
                        </p>
                        <div class="prose prose-invert max-w-none break-words">
                            {@html marked.parse(String(msg.content || ""))}
                        </div>

                        {#if msg.sources && msg.sources.length}
                            <button
                                class="mt-3 text-xs text-orange-400 hover:underline"
                                onclick={() =>
                                    (msg.showSources = !msg.showSources)}
                            >
                                {msg.showSources
                                    ? "📚 Hide Sources"
                                    : `📚 Sources (${msg.sources.length})`}
                            </button>

                            {#if msg.showSources}
                                <div class="mt-2 flex flex-col gap-2">
                                    {#each msg.sources as source, i}
                                        <a
                                            href={source}
                                            target="_blank"
                                            rel="noopener noreferrer"
                                            class="text-xs bg-white/10 px-2 py-1 rounded-md text-gray-300 hover:bg-white/20 transition"
                                        >
                                            {i + 1}. {new URL(source).hostname}
                                        </a>
                                    {/each}
                                </div>
                            {/if}
                        {/if}
                    </div>
                </div>
            {:else}
                <div class="flex justify-end">
                    <div
                        class="max-w-[80%] rounded-2xl rounded-tr-none bg-orange-600 p-4 text-white shadow-lg"
                    >
                        <div class="prose prose-invert max-w-none break-words">
                            {@html marked.parse(String(msg.content || ""))}
                        </div>
                    </div>
                </div>
            {/if}
        {/each}

        <!-- FIX: isTyping MUST be outside the #each loop -->
        {#if isTyping}
            <div class="flex justify-start">
                <div
                    class="bg-white/10 p-4 rounded-2xl rounded-tl-none border border-white/5"
                >
                    <div class="flex gap-1">
                        <div
                            class="w-2 h-2 bg-orange-400 rounded-full animate-bounce"
                        ></div>
                        <div
                            class="w-2 h-2 bg-orange-400 rounded-full animate-bounce [animation-delay:0.2s]"
                        ></div>
                        <div
                            class="w-2 h-2 bg-orange-400 rounded-full animate-bounce [animation-delay:0.4s]"
                        ></div>
                    </div>
                </div>
            </div>
        {/if}
    </div>
    <!-- 2. Input Box Area -->
    <div class="p-4 bg-black/20 border-t border-white/10">
        <div class="relative flex items-center">
            <input
                type="text"
                placeholder="Ask Adwa AI anything..."
                bind:value={input}
                onkeydown={(e) => e.key === "Enter" && sendMessage()}
                class="w-full bg-white/5 border border-white/10 rounded-2xl py-4 pl-6 pr-16 text-white placeholder:text-gray-400 focus:outline-none focus:ring-2 focus:ring-orange-500/50 transition-all"
            />

            <button
                onclick={sendMessage}
                class="absolute right-2 p-2 bg-orange-600 hover:bg-orange-500 text-white rounded-xl transition-all active:scale-95"
            >
                <!-- Send Icon -->
                <svg
                    xmlns="http://www.w3.org"
                    width="24"
                    height="24"
                    viewBox="0 0 24 24"
                    fill="none"
                    stroke="currentColor"
                    stroke-width="2"
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    ><line x1="22" y1="2" x2="11" y2="13"></line><polygon
                        points="22 2 15 22 11 13 2 9 22 2"
                    ></polygon></svg
                >
            </button>
        </div>
        <p class="text-[10px] text-center text-gray-500 mt-2 italic">
            Adwa AI can make mistakes. Check important info.
        </p>
    </div>
</div>










<style>
    .custom-scrollbar::-webkit-scrollbar {
        width: 6px;
    }
    .custom-scrollbar::-webkit-scrollbar-thumb {
        background: rgba(255, 255, 255, 0.1);
        border-radius: 10px;
    }

    :global(.prose ul) {
        list-style-type: disc;
        margin-left: 1.5rem;
    }
    :global(.prose ol) {
        list-style-type: decimal;
        margin-left: 1.5rem;
    }
    :global(.prose h1, .prose h2) {
        font-weight: bold;
        margin-top: 1rem;
    }
    :global(.prose p) {
        margin-bottom: 0.5rem;
    }
</style>
