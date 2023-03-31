<script lang="ts">
  import Router, { replace } from 'svelte-spa-router'
  import { wrap } from 'svelte-spa-router/wrap'
  import { get } from 'svelte/store'

  import Navbar from './lib/Navbar.svelte'
  import Sidebar from './lib/Sidebar.svelte'
  import Footer from './lib/Footer.svelte'
  import Home from './lib/Home.svelte'
  import Chat from './lib/Chat.svelte'
  import NewChat from './lib/NewChat.svelte'
  import Login from './lib/Login.svelte'
  import { accessToken, chatsStorage } from './lib/Storage.svelte'

  // Check if the API key is passed in as a "key" query parameter - if so, save it
  // Example: https://niek.github.io/chatgpt-web/#/?key=sk-...

// Define a variable to track whether the user is logged in
let loggedIn = false

// Check if the access token is already stored in local storage
  if (typeof get(accessToken) !== 'undefined' && get(accessToken) !== '') {
  // TODO validate login with server.
    console.log(get(accessToken))

    loggedIn = true
}

  // The definition of the routes with some conditions
  const routes = {
    '/': Home,

    '/login': Login,

    '/chat/new': wrap({
      component: NewChat,
      conditions: () => {
        return loggedIn
      }
    }),

    '/chat/:chatId': wrap({
      component: Chat,
      conditions: (detail) => {
        return loggedIn && $chatsStorage.find((chat) => chat.id === parseInt(detail?.params?.chatId as string)) !== undefined
      }
    }),

    '*': Home
  }

  // When the login form is submitted, set the loggedIn variable to true
  const handleLogin = () => {
    loggedIn = true
    replace('/')
  }
</script>

<Navbar />

<section class="section">
  <div class="container is-fullhd">
    <div class="columns">
      <div class="column is-one-fifth">
        <Sidebar />
      </div>
      <div class="column is-four-fifths" id="content">
        {#if loggedIn}
          <Router {routes} on:conditionsFailed={() => replace('/')}/>
        {:else}
          <Login on:login={handleLogin} />
        {/if}
      </div>
    </div>
  </div>
</section>
<Footer />
