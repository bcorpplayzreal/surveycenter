<script>
    import { onMount } from 'svelte';
    import { supabase } from '$lib/supabase';
  
    let email = $state("");
    let password = $state("");
    let user = $state(null);
    let message = $state("");
  
    async function loadUser() {
      const { data, error } = await supabase.auth.getUser();
  
      if (error) {
        console.log("getUser error:", error);
        user = null;
        return;
      }
  
      user = data.user;
    }
  
    async function signUp() {
      message = "";
  
      const { data, error } = await supabase.auth.signUp({
        email,
        password
      });
  
      if (error) {
        message = error.message;
        return;
      }
  
      console.log("signUp data:", data);
      message = "Sign up successful. Check your email if needed.";
      await loadUser();
    }
  
    async function signIn() {
      message = "";
  
      const { data, error } = await supabase.auth.signInWithPassword({
        email,
        password
      });
  
      if (error) {
        message = error.message;
        return;
      }
  
      user = data.user;
      message = "Signed in successfully.";
    }
  
    async function signOut() {
      await supabase.auth.signOut();
      user = null;
      message = "Signed out.";
    }
  
    onMount(() => {
      loadUser();
    });
  </script>
  
  <h1>SurveyCenter</h1>
  
  {#if user}
    <p>You are signed in as:</p>
    <strong>{user.email}</strong>
  
    <br />
    <br />
  
    <button onclick={signOut}>Sign out</button>
  {:else}
    <div>
      <label>
        Email
        <input bind:value={email} type="email" />
      </label>
    </div>
  
    <div>
      <label>
        Password
        <input bind:value={password} type="password" />
      </label>
    </div>
  
    <br />
  
    <button onclick={signUp}>Sign up</button>
    <button onclick={signIn}>Sign in</button>
  {/if}
  
  <p>{message}</p>