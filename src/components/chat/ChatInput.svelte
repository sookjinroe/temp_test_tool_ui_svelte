<script lang="ts">
  import { tick } from 'svelte';
  import { sessionStore } from '../../stores/sessions';
  import type { Session } from '../../stores/sessions';
  
  export let session: Session;
  export let onMessageSent: () => void = () => {};
  
  let messageInput = '';
  let currentSessionId = session.id;
  let textareaElement: HTMLTextAreaElement;
  
  // 세션이 변경되면 입력창 초기화
  $: if (session.id !== currentSessionId) {
    messageInput = '';
    currentSessionId = session.id;
  }
  
  async function sendMessage(messageContent: string) {
    if (!messageContent.trim() || !session || session.isLoading) return;
    
    const userMessage = messageContent.trim();
    const sessionId = session.id;
    
    // Clear input immediately
    messageInput = '';
    
    // Wait for DOM update and refocus
    await tick();
    if (textareaElement) {
      textareaElement.focus();
    }
    
    // Add user message
    sessionStore.addMessage(sessionId, {
      role: 'user',
      content: userMessage
    });
    
    onMessageSent();
    
    // Simulate AI response (replace with actual API call)
    sessionStore.updateSessionLoadingState(sessionId, true);
    
    setTimeout(() => {
      sessionStore.addMessage(sessionId, {
        role: 'assistant',
        content: `이것은 "${userMessage}"에 대한 시뮬레이션된 응답입니다. 실제 구현에서는 여기에 LLM API 호출이 들어갑니다.`
      });
      
      sessionStore.updateSessionLoadingState(sessionId, false);
      onMessageSent();
    }, 1000 + Math.random() * 2000);
  }
  
  async function handleKeydown(event: KeyboardEvent) {
    if (event.key === 'Enter' && !event.shiftKey) {
      event.preventDefault();
      
      // Check loading state before sending
      if (session?.isLoading) return;
      
      // Use the textarea's current value directly to avoid timing issues
      const currentValue = textareaElement.value;
      
      // Clear the textarea immediately
      textareaElement.value = '';
      messageInput = '';
      
      // Wait for DOM update
      await tick();
      
      // Send the message with the captured value
      await sendMessage(currentValue);
    }
  }
</script>

<div class="input-area">
  <div class="input-container">
    <div class="message-input-wrapper">
      <textarea
        bind:this={textareaElement}
        bind:value={messageInput}
        on:keydown={handleKeydown}
        placeholder="메시지를 입력하세요... (Shift+Enter로 줄바꿈)"
        class="message-input"
        rows="1"
      ></textarea>
      
      <button 
        class="send-button"
        on:click={() => sendMessage(messageInput)}
        disabled={!messageInput.trim() || session?.isLoading}
        title="메시지 전송"
      >
        <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
          <path d="M22 2L11 13M22 2l-7 20-4-9-9-4 20-7z"/>
        </svg>
      </button>
    </div>
  </div>
</div>

<style>
  .input-area {
    padding: var(--space-4);
    background: var(--vscode-panel-bg);
  }

  .input-container {
    max-width: 800px;
    margin: 0 auto;
  }

  .message-input-wrapper {
    display: flex;
    align-items: end;
    gap: var(--space-3);
    padding: var(--space-3);
    background: var(--vscode-bg-secondary);
    border: 1px solid var(--vscode-border);
    border-radius: 16px;
    transition: all 0.2s ease;
  }

  .message-input-wrapper:focus-within {
    border-color: var(--vscode-focus-border);
    box-shadow: 0 0 0 2px rgba(0, 125, 212, 0.1);
  }

  .message-input {
    flex: 1;
    min-height: 24px;
    max-height: 200px;
    border: none;
    background: transparent;
    color: var(--vscode-text-primary);
    font-size: 14px;
    font-family: inherit;
    resize: none;
    outline: none;
    line-height: 1.4;
  }

  .message-input::placeholder {
    color: var(--vscode-text-muted);
  }

  .send-button {
    width: 32px;
    height: 32px;
    border: none;
    border-radius: 8px;
    background: var(--vscode-button-bg);
    color: white;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s ease;
    flex-shrink: 0;
  }

  .send-button:hover:not(:disabled) {
    background: var(--vscode-button-hover);
    transform: translateY(-1px);
    box-shadow: var(--shadow-sm);
  }

  .send-button:disabled {
    opacity: 0.4;
    cursor: not-allowed;
  }
</style>