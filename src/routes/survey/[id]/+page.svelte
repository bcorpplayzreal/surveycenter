<script>
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabase';

  let survey = $state(null);
  let selectedAnswer = $state("");
  let message = $state("");
  let loading = $state(true);
  let alreadySubmitted = $state(false);
  let submitting = $state(false);

  function getOptions(survey) {
    return [
      survey.option_1,
      survey.option_2,
      survey.option_3,
      survey.option_4
    ];
  }

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
      message = "Please choose an answer before submitting.";
      return;
    }

    submitting = true;

    const { error } = await supabase
      .from('survey_responses')
      .insert({
        survey_id: survey.id,
        answer: selectedAnswer
      });

    submitting = false;

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
    <div class="top-badge">
      <div class="logo">SC</div>
      <span>SurveyCenter</span>
    </div>

    {#if loading}
      <div class="loading-box">
        <div class="spinner"></div>
        <p>Loading survey...</p>
      </div>
    {:else if survey}
      <div class="survey-header">
        <p class="eyebrow">Public Survey</p>
        <h1>{survey.title}</h1>
        <p class="question">{survey.question}</p>
      </div>

      {#if alreadySubmitted}
        <div class="thank-you">
          <div class="checkmark">✓</div>
          <h2>Response submitted</h2>
          <p>Thanks for answering this survey.</p>
        </div>
      {:else}
        <div class="options">
          {#each getOptions(survey) as option}
            <label class:selected={selectedAnswer === option} class="option-card">
              <input
                type="radio"
                bind:group={selectedAnswer}
                value={option}
              />

              <span class="radio-dot"></span>
              <span>{option}</span>
            </label>
          {/each}
        </div>

        <button
          class="primary-button"
          onclick={submitResponse}
          disabled={submitting}
        >
          {submitting ? "Submitting..." : "Submit Answer"}
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
      radial-gradient(circle at top left, #bfdbfe, transparent 32%),
      radial-gradient(circle at bottom right, #dbeafe, transparent 28%),
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
    background: rgba(255, 255, 255, 0.95);
    border: 1px solid rgba(226, 232, 240, 0.9);
    border-radius: 28px;
    padding: 34px;
    box-shadow: 0 24px 60px rgba(15, 23, 42, 0.18);
  }

  .top-badge {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 28px;
    color: #2563eb;
    font-weight: 800;
  }

  .logo {
    width: 42px;
    height: 42px;
    border-radius: 14px;
    background: #2563eb;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: bold;
  }

  .survey-header {
    text-align: center;
    margin-bottom: 28px;
  }

  .eyebrow {
    display: inline-block;
    margin: 0 0 12px;
    background: #eff6ff;
    color: #2563eb;
    border: 1px solid #bfdbfe;
    border-radius: 999px;
    padding: 6px 12px;
    font-size: 13px;
    font-weight: 800;
  }

  h1 {
    margin: 0;
    font-size: 34px;
    line-height: 1.1;
  }

  .question {
    margin: 14px 0 0;
    color: #475569;
    font-size: 18px;
    line-height: 1.5;
  }

  .options {
    display: flex;
    flex-direction: column;
    gap: 12px;
    margin-bottom: 22px;
  }

  .option-card {
    display: flex;
    align-items: center;
    gap: 12px;
    background: #f8fafc;
    border: 2px solid #e2e8f0;
    border-radius: 16px;
    padding: 16px;
    cursor: pointer;
    transition:
      border-color 0.15s ease,
      background 0.15s ease,
      transform 0.15s ease;
  }

  .option-card:hover {
    border-color: #93c5fd;
    background: #eff6ff;
    transform: translateY(-1px);
  }

  .option-card.selected {
    border-color: #2563eb;
    background: #eff6ff;
  }

  .option-card input {
    display: none;
  }

  .radio-dot {
    width: 18px;
    height: 18px;
    border: 2px solid #94a3b8;
    border-radius: 50%;
    flex-shrink: 0;
  }

  .option-card.selected .radio-dot {
    border: 6px solid #2563eb;
  }

  .option-card span:last-child {
    font-size: 16px;
    font-weight: 700;
    color: #334155;
  }

  button {
    border: none;
    border-radius: 14px;
    padding: 14px 16px;
    font-size: 16px;
    font-weight: 800;
    cursor: pointer;
    width: 100%;
  }

  .primary-button {
    background: #2563eb;
    color: white;
    box-shadow: 0 10px 20px rgba(37, 99, 235, 0.25);
  }

  .primary-button:hover {
    background: #1d4ed8;
  }

  .primary-button:disabled {
    background: #94a3b8;
    cursor: not-allowed;
    box-shadow: none;
  }

  .thank-you {
    background: #f0fdf4;
    border: 1px solid #bbf7d0;
    border-radius: 20px;
    padding: 26px;
    text-align: center;
  }

  .checkmark {
    width: 54px;
    height: 54px;
    margin: 0 auto 14px;
    border-radius: 50%;
    background: #16a34a;
    color: white;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 30px;
    font-weight: bold;
  }

  .thank-you h2 {
    margin: 0 0 8px;
  }

  .thank-you p {
    margin: 0;
    color: #166534;
  }

  .loading-box {
    text-align: center;
    padding: 30px 0;
    color: #64748b;
  }

  .spinner {
    width: 34px;
    height: 34px;
    margin: 0 auto 14px;
    border: 4px solid #dbeafe;
    border-top-color: #2563eb;
    border-radius: 50%;
    animation: spin 0.8s linear infinite;
  }

  @keyframes spin {
    to {
      transform: rotate(360deg);
    }
  }

  .message {
    margin-top: 18px;
    text-align: center;
    color: #2563eb;
    font-weight: 700;
  }

  .back-link {
    display: block;
    margin-top: 24px;
    text-align: center;
    color: #2563eb;
    font-weight: 800;
    text-decoration: none;
  }

  .back-link:hover {
    text-decoration: underline;
  }
</style>