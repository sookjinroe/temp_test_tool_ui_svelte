<script lang="ts">
  import type { Message, Session } from '../../stores/sessions';
  
  export let session: Session;
  export let chatContainer: HTMLElement | undefined = undefined;
  
  function formatTime(date: Date) {
    return date.toLocaleTimeString('ko-KR', { 
      hour: '2-digit', 
      minute: '2-digit' 
    });
  }
</script>

<div class="messages-container" bind:this={chatContainer}>
  <div class="messages">
    {#if session.messages.length === 0}
      <div class="empty-chat">
        <div class="empty-icon">
          <svg width="48" height="48" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
            <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
          </svg>
        </div>
        <h3>새로운 대화를 시작하세요</h3>
        <p>메시지를 입력하여 LLM과 대화를 시작할 수 있습니다.</p>
      </div>
    {:else}
      {#each session.messages as message (message.id)}
        <div class="message" class:user={message.role === 'user'} class:assistant={message.role === 'assistant'}>
          <div class="message-avatar">
            {#if message.role === 'user'}
              <div class="avatar user-avatar">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/>
                  <circle cx="12" cy="7" r="4"/>
                </svg>
              </div>
            {:else}
              <div class="avatar assistant-avatar">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
                  <path d="M12 8V4H8"/>
                  <rect width="16" height="12" x="4" y="8" rx="2"/>
                  <path d="M2 14h2"/>
                  <path d="M20 14h2"/>
                  <path d="M15 13v2"/>
                  <path d="M9 13v2"/>
                </svg>
              </div>
            {/if}
          </div>
          
          <div class="message-content">
            <div class="message-header">
              <span class="message-role">
                {message.role === 'user' ? '사용자' : 'Assistant'}
              </span>
              <span class="message-time">{formatTime(message.timestamp)}</span>
            </div>
            
            <div class="message-text">
              {message.content}
            </div>
          </div>
        </div>
      {/each}
    {/if}
    
    {#if session.isLoading}
      <div class="message assistant">
        <div class="message-avatar">
          <div class="avatar assistant-avatar">
            <svg width="16" height="16" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
              <path d="M12 8V4H8"/>
              <rect width="16" height="12" x="4" y="8" rx="2"/>
              <path d="M2 14h2"/>
              <path d="M20 14h2"/>
              <path d="M15 13v2"/>
              <path d="M9 13v2"/>
            </svg>
          </div>
        </div>
        <div class="message-content">
          <div class="message-header">
            <span class="message-role">Assistant</span>
          </div>
          <div class="typing-indicator">
            <div class="typing-dots">
              <div class="dot"></div>
              <div class="dot"></div>
              <div class="dot"></div>
            </div>
          </div>
        </div>
      </div>
    {/if}
  </div>
</div>

<style>
  .messages-container {
    flex: 1;
    overflow-y: auto;
    padding: var(--space-4);
  }

  .messages {
    max-width: 800px;
    margin: 0 auto;
  }

  .empty-chat {
    text-align: center;
    padding: var(--space-8) var(--space-4);
    color: var(--vscode-text-muted);
  }

  .empty-icon {
    margin-bottom: var(--space-4);
    opacity: 0.5;
  }

  .empty-chat h3 {
    font-size: 18px;
    font-weight: 500;
    color: var(--vscode-text-primary);
    margin-bottom: var(--space-2);
  }

  .empty-chat p {
    font-size: 14px;
    color: var(--vscode-text-secondary);
  }

  .message {
    display: flex;
    gap: var(--space-3);
    margin-bottom: var(--space-6);
  }

  .message-avatar {
    flex-shrink: 0;
  }

  .avatar {
    width: 32px;
    height: 32px;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 12px;
    font-weight: 500;
  }

  .user-avatar {
    background: var(--vscode-accent);
    color: white;
  }

  .assistant-avatar {
    background: var(--vscode-bg-tertiary);
    color: var(--vscode-text-primary);
    border: 1px solid var(--vscode-border);
  }

  .message-content {
    flex: 1;
    min-width: 0;
  }

  .message-header {
    display: flex;
    align-items: center;
    gap: var(--space-2);
    margin-bottom: var(--space-2);
  }

  .message-role {
    font-size: 12px;
    font-weight: 600;
    color: var(--vscode-text-secondary);
  }

  .message-time {
    font-size: 11px;
    color: var(--vscode-text-muted);
  }

  .message-text {
    font-size: 14px;
    line-height: 1.5;
    color: var(--vscode-text-primary);
    white-space: pre-wrap;
    word-wrap: break-word;
  }

  .typing-indicator {
    padding: var(--space-3) 0;
  }

  .typing-dots {
    display: flex;
    gap: 4px;
  }

  .dot {
    width: 6px;
    height: 6px;
    border-radius: 50%;
    background: var(--vscode-text-muted);
    animation: typing 1.4s infinite ease-in-out;
  }

  .dot:nth-child(1) { animation-delay: -0.32s; }
  .dot:nth-child(2) { animation-delay: -0.16s; }

  @keyframes typing {
    0%, 80%, 100% {
      opacity: 0.3;
      transform: scale(0.8);
    }
    40% {
      opacity: 1;
      transform: scale(1);
    }
  }
</style>