<script lang="ts">
  import { params, replace } from 'svelte-spa-router'
  import { Auth } from 'aws-amplify'

  import { loggedIn, chatsStorage, clearChats, deleteChat } from './Storage.svelte'
  import { exportAsMarkdown } from './Export.svelte'

  $: sortedChats = $chatsStorage.sort((a, b) => b.id - a.id)

  $: activeChatId = $params && $params.chatId ? parseInt($params.chatId) : undefined

  function delChat (chatId) {
    if (activeChatId === chatId) {
    // Find the max chatId other than the current one
      const newChatId = sortedChats.reduce((maxId, chat) => {
        if (chat.id === chatId) return maxId
        return Math.max(maxId, chat.id)
      }, 0)

      if (!newChatId) {
        // No other chats, clear all and go to home
        replace('/').then(() => { deleteChat(chatId) })
      } else {
        // Delete the current chat and go to the max chatId
        replace(`/chat/${newChatId}`).then(() => { deleteChat(chatId) })
      }
    } else {
      deleteChat(chatId)
    }
  }
  const handleLogout = async () => {
    try {
      await Auth.signOut()
      $loggedIn = false
      replace('/login')
    } catch (error) {
      console.error('Error signing out:', error)
    }
  }
</script>

<aside class="menu"  class:is-hidden={!$loggedIn}>
  <p class="menu-label">Chats</p>
  <ul class="menu-list">
    {#if sortedChats.length === 0}
      <li><a href={'#'} class="is-disabled">No chats yet...</a></li>
    {:else}
      <li>
        <ul>
          {#each sortedChats as chat}
            <li>
              <a style="position: relative" href={`#/chat/${chat.id}`}  class:is-active={activeChatId === chat.id}>
                <a class="is-pulled-right is-hidden px-1 py-0 greyscale has-text-weight-bold delete-button" href={'$'} on:click|preventDefault={() => delChat(chat.id)}>🗑️</a>
                {chat.name || `Chat ${chat.id}`}
              </a>
            </li>
          {/each}
        </ul>
      </li>
    {/if}
  </ul>
  <p class="menu-label">Actions</p>
  <ul class="menu-list">

    <li>
      <a href={'#/chat/new'} class="panel-block"
        ><span class="greyscale mr-2">➕</span> New chat</a
      >
    </li>
    <li>
      <a class="panel-block"
        href="{'#/'}"
      
        on:click|preventDefault={() => {
          const confirmDelete = window.confirm('Are you sure you want to delete all your chats?')
          if (confirmDelete) {
            replace('#/').then(() => clearChats())
          }
        }}><span class="greyscale mr-2">🗑️</span> Clear chats</a
      >
    </li>
    {#if activeChatId}
      <li>
        <a
          href={'#/'}
          class="panel-block"
          class:is-disabled={!$loggedIn}
          on:click|preventDefault={() => {
            if (activeChatId) {
              exportAsMarkdown(activeChatId)
            }
          }}><span class="greyscale mr-2">📥</span> Export chat</a
        >
      </li>
    {/if}
    {#if $loggedIn}
    <li>
      <a
        href={'#/'}
        class="panel-block"
        class:is-disabled={!$loggedIn}
        on:click={handleLogout}><span class="greyscale mr-2">👋</span>Logout</a
      >
    </li>
  {/if}
  </ul>
</aside>
