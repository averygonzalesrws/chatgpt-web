<script lang="ts">
  import { loggedIn } from './Storage.svelte'

</script>

<article class="message">
  <div class="message-body">
    <strong>BanterBuddy</strong>
    is a simple one-page web interface to the OpenAI ChatGPT API.  OpenAI bills per token (usage-based), which means it is a lot cheaper than
    <a href="https://openai.com/blog/chatgpt-plus" target="_blank" rel="noreferrer">ChatGPT Plus</a>, unless you use
    more than 10 million tokens per month. All messages are stored in your browser's local storage, so everything is
    <strong>private</strong>. You can also close the browser tab and come back later to continue the conversation.
  </div>
</article>

{#if $loggedIn}
  <article class="message is-info">
    <div class="message-body">
      Select an existing chat on the sidebar, or
      <a href={'#/chat/new'}>create a new chat</a>
    </div>
  </article>
{/if}
