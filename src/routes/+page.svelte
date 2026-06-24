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
      message = "Account created successfully.";
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
  
  <main class="page">
    <section class="card">
      <div class="header">
        <div class="logo">SC</div>
        <h1>SurveyCenter</h1>
        <p>Create and share simple surveys.</p>
      </div>
  
      {#if user}
        <div class="signed-in-box">
          <p>You are signed in as</p>
          <strong>{user.email}</strong>
        </div>
  
        <button class="secondary-button" onclick={signOut}>
          Sign out
        </button>
      {:else}
        <div class="form">
          <label>
            Email
            <input
              bind:value={email}
              type="email"
              placeholder="you@example.com"
            />
          </label>
  
          <label>
            Password
            <input
              bind:value={password}
              type="password"
              placeholder="Enter your password"
            />
          </label>
  
          <button class="primary-button" onclick={signIn}>
            Sign in
          </button>
  
          <button class="secondary-button" onclick={signUp}>
            Create account
          </button>
        </div>
      {/if}
  
      {#if message}
        <p class="message">{message}</p>
      {/if}
    </section>
  </main>
  
  <style>
    :global(body) {
      margin: 0;
      font-family: Arial, Helvetica, sans-serif;
      background:
        radial-gradient(circle at top left, #dbeafe, transparent 30%),
        linear-gradient(135deg, #f8fafc, #e0f2fe);
      color: #0f172a;
    }
  
    .page {
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 24px;
    }
  
    .card {
      width: 100%;
      max-width: 420px;
      background: white;
      border-radius: 24px;
      padding: 32px;
      box-shadow: 0 20px 50px rgba(15, 23, 42, 0.15);
    }
  
    .header {
      text-align: center;
      margin-bottom: 28px;
    }
  
    .logo {
      width: 56px;
      height: 56px;
      margin: 0 auto 16px;
      border-radius: 16px;
      background: #2563eb;
      color: white;
      display: flex;
      align-items: center;
      justify-content: center;
      font-weight: bold;
      font-size: 20px;
    }
  
    h1 {
      margin: 0;
      font-size: 32px;
    }
  
    .header p {
      margin: 8px 0 0;
      color: #64748b;
    }
  
    .form {
      display: flex;
      flex-direction: column;
      gap: 16px;
    }
  
    label {
      display: flex;
      flex-direction: column;
      gap: 6px;
      font-size: 14px;
      font-weight: 600;
      color: #334155;
    }
  
    input {
      padding: 12px 14px;
      border: 1px solid #cbd5e1;
      border-radius: 12px;
      font-size: 16px;
    }
  
    input:focus {
      outline: none;
      border-color: #2563eb;
      box-shadow: 0 0 0 3px rgba(37, 99, 235, 0.15);
    }
  
    button {
      border: none;
      border-radius: 12px;
      padding: 12px 14px;
      font-size: 16px;
      font-weight: bold;
      cursor: pointer;
    }
  
    .primary-button {
      background: #2563eb;
      color: white;
      margin-top: 8px;
    }
  
    .primary-button:hover {
      background: #1d4ed8;
    }
  
    .secondary-button {
      background: #e2e8f0;
      color: #0f172a;
    }
  
    .secondary-button:hover {
      background: #cbd5e1;
    }
  
    .signed-in-box {
      background: #f1f5f9;
      border-radius: 16px;
      padding: 18px;
      margin-bottom: 16px;
      text-align: center;
    }
  
    .signed-in-box p {
      margin: 0 0 6px;
      color: #64748b;
    }
  
    .message {
      margin-top: 18px;
      text-align: center;
      color: #2563eb;
      font-weight: 600;
    }
  </style>