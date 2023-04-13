<script lang="ts">
  import Router, { location, replace } from 'svelte-spa-router'
  import { wrap } from 'svelte-spa-router/wrap'
  import { onMount } from 'svelte'
  import { Auth, Hub } from 'aws-amplify'
  import Navbar from './lib/Navbar.svelte'
  import Sidebar from './lib/Sidebar.svelte'
  import Footer from './lib/Footer.svelte'
  import Home from './lib/Home.svelte'
  import Chat from './lib/Chat.svelte'
  import NewChat from './lib/NewChat.svelte'
  import Login from './lib/Login.svelte'
  import { chatsStorage, loggedIn } from './lib/Storage.svelte'

  // Check if the API key is passed in as a "key" query parameter - if so, save it
  // Example: https://niek.github.io/chatgpt-web/#/?key=sk-...
  // Amplify logic
  onMount(() => {
    Hub.listen('auth', ({ payload: { event, data } }) => {
      console.log(event)

      switch (event) {
        case 'signIn':
        case 'confirmSignUp':
        case 'cognitoHostedUI':
          handleLogin() // Call handleLogin function when the user signs in
          break
        case 'signOut':
          $loggedIn = false
          break
        case 'signIn_failure':
        case 'cognitoHostedUI_failure':
          console.log(data)
          console.log('Sign in failure', data)
          break
      }
    })

    Auth.currentAuthenticatedUser()
      .then(() => handleLogin())
      .catch(() => console.log('Not signed in'))
  })

// Define a variable to track whether the user is logged in


  // The definition of the routes with some conditions
  const routes = {
    '/': Home,

    '/login': Login,

    '/chat/new': wrap({
      component: NewChat,
      conditions: () => {
        return $loggedIn
      }
    }),

    '/chat/:chatId': wrap({
      component: Chat,
      conditions: (detail) => {
        return $loggedIn && $chatsStorage.find((chat) => chat.id === parseInt(detail?.params?.chatId as string)) !== undefined
      }
    }),

    '*': Home
  }

  // When the login form is submitted, set the loggedIn variable to true
  const handleLogin = () => {
    $loggedIn = true
    replace('/')
  }

</script>

<Navbar />

<section class="section" >

  <div class="container is-fullhd">
    <div class="columns">
      <div class="column is-one-fifth">
        <Sidebar />
      </div>
      <div class="column is-four-fifths" id="content">
        {#key $location}  
          {#if $loggedIn}
            <Router {routes} on:conditionsFailed={() => replace('/')}/>
          {:else}
            <Login on:login={handleLogin} />
          {/if}
        {/key}
      </div>
    </div>
  </div>
</section>
<Footer />
