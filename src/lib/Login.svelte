<script lang="ts">
  import { Auth } from "aws-amplify";

  let username = "";
  let password = "";
  let confirmationCode = "";
  let showConfirmationForm = false;
  let errorMessage = "";

  function isValidEmployeeEmail (str: string): boolean {
    return (/^(?=.*@(retailerwebservices\.com|nationwidegroup\.org)$).+$/i.test(username))
  }

  const signIn = async (username: string, password: string) => {
    try {
      // Validate email address
      if (!isValidEmployeeEmail(username)) {
        errorMessage = 'Invalid email address. Please use a valid @retailerwebservices.com or @nationwidegroup.org email address.'
        return;
      }

      console.log(username, password);
      const user = await Auth.signIn(username, password);
      console.log(user);
      return user;
    } catch (error) {
      console.error(error);
      if (error.code === "UserNotConfirmedException") {
        showConfirmationForm = true;
      } else {
        errorMessage = error;
      }
    }
  };
  const signUp = async (username: string, password: string) => {
    try {
      if (!isValidEmployeeEmail(username)) {
        errorMessage = 'Invalid email address. Please use a valid @retailerwebservices.com or @nationwidegroup.org email address.'
        return;
      }
      const { user } = await Auth.signUp({
        username,
        password,
        attributes: {
          email: username
        }
      });
      console.log(user);
      showConfirmationForm = true;
    } catch (error) {
      console.error(error);
      errorMessage = error;
    }
  };

  const confirmSignUp = async (username: string, confirmationCode: string) => {
    try {
      await Auth.confirmSignUp(username, confirmationCode);
      showConfirmationForm = false;
    } catch (error) {
      console.error(error);
      errorMessage = error;
    }
  };
</script>

<div class="card p-5">
  {#if errorMessage}
    <div class="notification is-danger">
      {errorMessage}
    </div>
  {/if}

  {#if showConfirmationForm}
    <form
      on:submit|preventDefault={() => confirmSignUp(username, confirmationCode)}
    >
      <div class="field">
        <label class="label">Email Confirmation Code</label>
        <div class="control">
          <input
            class="input"
            type="text"
            placeholder="Confirmation code"
            bind:value={confirmationCode}
            required
          />
        </div>
      </div>
      <div class="field">
        <div class="control">
          <button class="button is-primary" type="submit"
            >Confirm Sign Up</button
          >
        </div>
      </div>
    </form>
  {:else}
    <form on:submit|preventDefault={() => signIn(username, password)}>
      <div class="field">
        <label class="label">Email</label>
        <div class="control">
          <input
            class="input"
            type="email"
            placeholder="Email"
            bind:value={username}
            required
          />
        </div>
      </div>

      <div class="field">
        <label class="label">Password</label>
        <div class="control">
          <input
            class="input"
            type="password"
            placeholder="Password"
            bind:value={password}
            required
          />
        </div>
      </div>

      <div class="field is-grouped">
        <div class="control">
          <button
            class="button is-primary"
            type="button"
            on:click={() => signUp(username, password)}>Sign Up</button
          >
        </div>
        <div class="control">
          <button class="button is-primary" type="submit">Login</button>
        </div>
      </div>
    </form>
  {/if}
</div>
