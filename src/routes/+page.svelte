<script>
    import { onMount } from 'svelte';
    import { supabase } from '$lib/supabase';
  
    let email = $state("");
    let password = $state("");
    let user = $state(null);
    let message = $state("");
  
    let surveys = $state([]);
  
    let title = $state("");
    let question = $state("");
    let option1 = $state("");
    let option2 = $state("");
    let option3 = $state("");
    let option4 = $state("");
  
    async function loadUser() {
      const { data, error } = await supabase.auth.getUser();
  
      if (error) {
        console.log("getUser error:", error);
        user = null;
        return;
      }
  
      user = data.user;
  
      if (user) {
        await loadSurveys();
      }
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
      await loadSurveys();
    }
  
    async function signOut() {
      await supabase.auth.signOut();
      user = null;
      surveys = [];
      message = "Signed out.";
    }
  
    async function loadSurveys() {
      if (!user) return;
  
      const { data, error } = await supabase
        .from('surveys')
        .select('*')
        .eq('owner_id', user.id)
        .order('created_at', { ascending: false });
  
      if (error) {
        console.log("loadSurveys error:", error);
        message = error.message;
        return;
      }
  
      surveys = data;
    }
  
    async function createSurvey() {
      message = "";
  
      if (!title || !question || !option1 || !option2 || !option3 || !option4) {
        message = "Please fill out all survey fields.";
        return;
      }
  
      const { error } = await supabase
        .from('surveys')
        .insert({
          title,
          question,
          option_1: option1,
          option_2: option2,
          option_3: option3,
          option_4: option4
        });
  
      if (error) {
        console.log("createSurvey error:", error);
        message = error.message;
        return;
      }
  
      message = "Survey created successfully.";
  
      title = "";
      question = "";
      option1 = "";
      option2 = "";
      option3 = "";
      option4 = "";
  
      await loadSurveys();
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
  
        <div class="survey-form">
          <h2>Create a Survey</h2>
  
          <label>
            Survey title
            <input
              bind:value={title}
              placeholder="Favorite Programming Language"
            />
          </label>
  
          <label>
            Question
            <input
              bind:value={question}
              placeholder="Which language do you like most?"
            />
          </label>
  
          <label>
            Option 1
            <input bind:value={option1} placeholder="Python" />
          </label>
  
          <label>
            Option 2
            <input bind:value={option2} placeholder="JavaScript" />
          </label>
  
          <label>
            Option 3
            <input bind:value={option3} placeholder="C++" />
          </label>
  
          <label>
            Option 4
            <input bind:value={option4} placeholder="Java" />
          </label>
  
          <button class="primary-button" onclick={createSurvey}>
            Create Survey
          </button>
        </div>
  
        <div class="my-surveys">
          <h2>My Surveys</h2>
  
          {#if surveys.length === 0}
            <p class="empty-text">No surveys yet.</p>
          {:else}
            {#each surveys as survey}
              <div class="survey-item">
                <h3>{survey.title}</h3>
                <p>{survey.question}</p>
  
                <ul>
                  <li>{survey.option_1}</li>
                  <li>{survey.option_2}</li>
                  <li>{survey.option_3}</li>
                  <li>{survey.option_4}</li>
                </ul>
              </div>
            {/each}
          {/if}
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
      max-width: 520px;
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
  
    h2 {
      margin-top: 0;
      font-size: 22px;
    }
  
    .header p {
      margin: 8px 0 0;
      color: #64748b;
    }
  
    .form,
    .survey-form {
      display: flex;
      flex-direction: column;
      gap: 16px;
    }
  
    .survey-form {
      margin-top: 24px;
      padding-top: 24px;
      border-top: 1px solid #e2e8f0;
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
      width: 100%;
      background: #e2e8f0;
      color: #0f172a;
      margin-top: 18px;
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
  
    .my-surveys {
      margin-top: 28px;
      padding-top: 24px;
      border-top: 1px solid #e2e8f0;
    }
  
    .survey-item {
      background: #f8fafc;
      border: 1px solid #e2e8f0;
      border-radius: 16px;
      padding: 16px;
      margin-bottom: 14px;
    }
  
    .survey-item h3 {
      margin: 0 0 8px;
    }
  
    .survey-item p {
      margin: 0 0 8px;
      color: #475569;
    }
  
    .survey-item ul {
      margin: 0;
      padding-left: 20px;
      color: #334155;
    }
  
    .empty-text {
      color: #64748b;
    }
  
    .message {
      margin-top: 18px;
      text-align: center;
      color: #2563eb;
      font-weight: 600;
    }
  </style>