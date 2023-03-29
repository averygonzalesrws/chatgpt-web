<!-- Login.svelte -->
<script>
  import { createEventDispatcher } from 'svelte'
  import { writable } from 'svelte/store'
  import { accessToken } from './Storage.svelte'

  let email = ''
  let password = ''
  const dispatch = createEventDispatcher()

  async function handleSubmit (event) {
    event.preventDefault()

    try {
      const user = await login(email, password)

      // User is authenticated, navigate to a new page
    } catch (error) {
      // Display an error message
      console.log(error)
    }
  }


  const DEFAULT_STATE = {
    authenticated: false,
    token: null,
    user: null
  }

  // A writable store that holds the authentication state
  export const authStore = writable(DEFAULT_STATE)

  // A function to update the authentication state
  function updateAuth (authenticated, token, user) {
    authStore.set({ authenticated, token, user })
  }

  // A function to authenticate the user
  async function login (username, password) {
    // Update the authentication state
    updateAuth(true, 'abc123', username)

    // Store the access token in local storage
    accessToken.set('abc123')

    dispatch('login', true)

    /* try {
      // Call the login API endpoint to obtain an access token
      const response = await fetch('/api/login', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ username, password })
      })

      if (!response.ok) {
        throw new Error(`Failed to log in: ${response.statusText}`)
      }

      const { token, user } = await response.json()

      // Update the authentication state
      updateAuth(true, token, user)

      // Store the access token in local storage
      accessToken.set('accessToken', token)
    } catch (error) {
      console.error(error)
      throw error
    } */
  }

  // A function to log out the user
  function logout () {
    // Update the authentication state
    updateAuth(false, null, null)

    // Remove the access token from local storage
    accessToken.removeItem('accessToken')
  }

  // A function to check if the user is authenticated
  function isAuthenticated () {
    const { authenticated, token } = $authStore
    return authenticated && token != null
  }

  // A function to get the user object
  function getUser () {
    const { user } = $authStore
    return user
  }
</script>

<div class="card p-5">
  <form on:submit={handleSubmit}>
    <div class="field">
      <label class="label">Email</label>
      <div class="control">
        <input class="input" type="email" placeholder="Email" bind:value={email} required />
      </div>
    </div>

    <div class="field">
      <label class="label">Password</label>
      <div class="control">
        <input class="input" type="password" placeholder="Password" bind:value={password} required />
      </div>
    </div>

    <div class="field">
      <div class="control">
        <button class="button is-primary" type="submit">Login</button>
      </div>
    </div>
  </form>
</div>
