<script>
    import { onMount } from 'svelte';
    import { supabase } from '$lib/supabase';
  
    let survey = $state(null);
    let selectedAnswer = $state("");
    let message = $state("");
    let loading = $state(true);
    let alreadySubmitted = $state(false);
  
    async function loadSurvey() {
      const surveyId = window.location.pathname.split('/').pop();
  
      const { data, error } = await supabase
        .from('surveys')
        .select('*')
        .eq('id', surveyId)
        .eq('is_published', true)
        .single();
  
      if (error) {
        console.log("loadSurvey error:", error);
        message = "Survey not found.";
        loading = false;
        return;
      }
  
      survey = data;
      loading = false;
    }
  
    async function submitResponse() {
      message = "";
  
      if (!selectedAnswer) {
        message = "Please choose an answer.";
        return;
      }
  
      const { error } = await supabase
        .from('survey_responses')
        .insert({
          survey_id: survey.id,
          answer: selectedAnswer
        });
  
      if (error) {
        console.log("submitResponse error:", error);
        message = error.message;
        return;
      }
  
      alreadySubmitted = true;
      message = "Thank you for your response!";
    }
  
    onMount(() => {
      loadSurvey();
    });
  </script>
  
  <main class="page">
    <section class="card">
      <div class="header">
        <div class="logo">SC</div>
        <h1>SurveyCenter</h1>
        <p>Public survey</p>
      </div>
  
      {#if loading}
        <p>Loading survey...</p>
      {:else if survey}
        <h2>{survey.title}</h2>
        <p class="question">{survey.question}</p>
  
        {#if alreadySubmitted}
          <div class="thank-you">
            <h3>Response submitted ✅</h3>
            <p>Thanks for answering this survey.</p>
          </div>
        {:else}
          <div class="options">
            <label>
              <input
                type="radio"
                bind:group={selectedAnswer}
                value={survey.option_1}
              />
              {survey.option_1}
            </label>
  
            <label>
              <input
                type="radio"
                bind:group={selectedAnswer}
                value={survey.option_2}
              />
              {survey.option_2}
            </label>
  
            <label>
              <input
                type="radio"
                bind:group={selectedAnswer}
                value={survey.option_3}
              />
              {survey.option_3}
            </label>
  
            <label>
              <input
                type="radio"
                bind:group={selectedAnswer}
                value={survey.option_4}
              />
              {survey.option_4}
            </label>
          </div>
  
          <button class="primary-button" onclick={submitResponse}>
            Submit Answer
          </button>
        {/if}
      {/if}
  
      {#if message}
        <p class="message">{message}</p>
      {/if}
  
      <a class="back-link" href="/">
        Back to SurveyCenter
      </a>
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
      max-width: 480px;
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
      margin-bottom: 10px;
    }
  
    .header p {
      margin: 8px 0 0;
      color: #64748b;
    }
  
    .question {
      color: #475569;
      font-size: 18px;
      margin-bottom: 20px;
    }
  
    .options {
      display: flex;
      flex-direction: column;
      gap: 12px;
      margin-bottom: 20px;
    }
  
    label {
      display: flex;
      align-items: center;
      gap: 10px;
      background: #f8fafc;
      border: 1px solid #e2e8f0;
      border-radius: 14px;
      padding: 14px;
      cursor: pointer;
    }
  
    input[type="radio"] {
      width: 18px;
      height: 18px;
    }
  
    button {
      border: none;
      border-radius: 12px;
      padding: 12px 14px;
      font-size: 16px;
      font-weight: bold;
      cursor: pointer;
      width: 100%;
    }
  
    .primary-button {
      background: #2563eb;
      color: white;
    }
  
    .primary-button:hover {
      background: #1d4ed8;
    }
  
    .thank-you {
      background: #f0fdf4;
      border: 1px solid #bbf7d0;
      border-radius: 16px;
      padding: 18px;
      text-align: center;
    }
  
    .message {
      margin-top: 18px;
      text-align: center;
      color: #2563eb;
      font-weight: 600;
    }
  
    .back-link {
      display: block;
      margin-top: 22px;
      text-align: center;
      color: #2563eb;
      font-weight: bold;
      text-decoration: none;
    }
  
    .back-link:hover {
      text-decoration: underline;
    }
  </style>