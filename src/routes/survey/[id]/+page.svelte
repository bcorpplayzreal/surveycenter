<script>
  import { onMount } from 'svelte';
  import { supabase } from '$lib/supabase';

  let survey = $state(null);
  let questions = $state([]);
  let selectedAnswers = $state({});
  let message = $state("");
  let loading = $state(true);
  let alreadySubmitted = $state(false);
  let submitting = $state(false);

  function getOptions(question) {
    return [
      question.option_1,
      question.option_2,
      question.option_3,
      question.option_4
    ];
  }

  function updateAnswer(questionId, answer) {
    selectedAnswers = {
      ...selectedAnswers,
      [questionId]: answer
    };

    message = "";
  }

  function allQuestionsAnswered() {
    return questions.every((question) => selectedAnswers[question.id]);
  }

  async function loadSurvey() {
    const surveyId = window.location.pathname.split('/').pop();

    const { data: surveyData, error: surveyError } = await supabase
      .from('surveys')
      .select('*')
      .eq('id', surveyId)
      .eq('is_published', true)
      .single();

    if (surveyError) {
      console.log("loadSurvey error:", surveyError);
      message = "Survey not found.";
      loading = false;
      return;
    }

    survey = surveyData;

    const { data: questionData, error: questionError } = await supabase
      .from('survey_questions')
      .select('*')
      .eq('survey_id', surveyId)
      .order('order_num', { ascending: true });

    if (questionError) {
      console.log("loadQuestions error:", questionError);
      message = "Could not load survey questions.";
      loading = false;
      return;
    }

    questions = questionData;
    loading = false;
  }

  async function submitResponse() {
  message = "";

  if (!allQuestionsAnswered()) {
    message = "Please answer every question before submitting.";
    return;
  }

  submitting = true;

  const firstQuestion = questions[0];
  const firstAnswer = selectedAnswers[firstQuestion.id];

  const responseToSave = {
    survey_id: survey.id,
    answer: firstAnswer,
    answers: { ...selectedAnswers }
  };

  console.log("Saving response:", responseToSave);

  const { data, error } = await supabase
    .from('survey_responses')
    .insert(responseToSave)
    .select();

  submitting = false;

  if (error) {
    console.log("submitResponse error:", error);
    message = error.message;
    return;
  }

  console.log("Saved response:", data);

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
        <p class="question-count">
          {questions.length} question{questions.length === 1 ? "" : "s"}
        </p>
      </div>

      {#if alreadySubmitted}
        <div class="thank-you">
          <div class="checkmark">✓</div>
          <h2>Response submitted</h2>
          <p>Thanks for answering this survey.</p>
        </div>
      {:else}
        <div class="questions">
          {#each questions as question, index}
            <div class="question-card">
              <h2>Question {index + 1}</h2>
              <p class="question-text">{question.question_text}</p>

              <div class="options">
                {#each getOptions(question) as option}
                  <label
                    class={`option-card ${selectedAnswers[question.id] === option ? "selected" : ""}`}
                  >
                    <input
                      type="radio"
                      name={question.id}
                      checked={selectedAnswers[question.id] === option}
                      onchange={() => updateAnswer(question.id, option)}
                    />

                    <span class="radio-dot"></span>
                    <span>{option}</span>
                  </label>
                {/each}
              </div>
            </div>
          {/each}
        </div>

        <button
          class="primary-button"
          onclick={submitResponse}
          disabled={submitting}
        >
          {submitting ? "Submitting..." : "Submit Survey"}
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
    align-items: flex-start;
    justify-content: center;
    padding: 40px 24px;
  }

  .card {
    width: 100%;
    max-width: 760px;
    background: white;
    border: 3px solid #0f172a;
    border-radius: 10px;
    padding: 34px;
    box-shadow: 12px 12px 0 rgba(15, 23, 42, 0.18);
  }

  .top-badge {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 28px;
    color: #2563eb;
    font-weight: 900;
  }

  .logo {
    width: 44px;
    height: 44px;
    border-radius: 8px;
    background: #2563eb;
    color: white;
    border: 3px solid #0f172a;
    box-shadow: 4px 4px 0 #facc15;
    display: flex;
    align-items: center;
    justify-content: center;
    font-weight: 900;
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
    border: 2px solid #bfdbfe;
    border-radius: 999px;
    padding: 6px 12px;
    font-size: 13px;
    font-weight: 900;
  }

  h1 {
    margin: 0;
    font-size: 36px;
    line-height: 1.1;
  }

  .question-count {
    margin: 12px 0 0;
    color: #64748b;
    font-weight: 700;
  }

  .questions {
    display: flex;
    flex-direction: column;
    gap: 18px;
    margin-bottom: 22px;
  }

  .question-card {
    background: #f8fafc;
    border: 3px solid #0f172a;
    border-radius: 8px;
    padding: 20px;
    box-shadow: 6px 6px 0 #dbeafe;
  }

  .question-card h2 {
    margin: 0 0 8px;
    color: #2563eb;
    font-size: 20px;
  }

  .question-text {
    margin: 0 0 16px;
    color: #334155;
    font-size: 18px;
    font-weight: 800;
  }

  .options {
    display: flex;
    flex-direction: column;
    gap: 12px;
  }

  .option-card {
    display: flex;
    align-items: center;
    gap: 12px;
    background: white;
    border: 3px solid #0f172a;
    border-radius: 8px;
    padding: 14px;
    cursor: pointer;
    box-shadow: 4px 4px 0 #e2e8f0;
  }

  .option-card:hover {
    background: #eff6ff;
  }

  .option-card.selected {
    background: #eff6ff;
    box-shadow: 4px 4px 0 #93c5fd;
  }

  .option-card input {
    display: none;
  }

  .radio-dot {
    width: 18px;
    height: 18px;
    border: 3px solid #0f172a;
    border-radius: 50%;
    background: white;
    flex-shrink: 0;
  }

  .option-card.selected .radio-dot {
    background: #2563eb;
    box-shadow: inset 0 0 0 4px white;
  }

  .option-card span:last-child {
    font-size: 16px;
    font-weight: 800;
    color: #334155;
  }

  button {
    border: 3px solid #0f172a;
    border-radius: 8px;
    padding: 14px 16px;
    font-size: 16px;
    font-weight: 900;
    cursor: pointer;
    width: 100%;
    box-shadow: 5px 5px 0 #0f172a;
  }

  .primary-button {
    background: #2563eb;
    color: white;
  }

  .primary-button:hover {
    transform: translate(1px, 1px);
    box-shadow: 4px 4px 0 #0f172a;
  }

  .primary-button:disabled {
    background: #94a3b8;
    cursor: not-allowed;
    box-shadow: none;
  }

  .thank-you {
    background: #f0fdf4;
    border: 3px solid #166534;
    border-radius: 8px;
    padding: 26px;
    text-align: center;
    box-shadow: 6px 6px 0 #bbf7d0;
  }

  .checkmark {
    width: 54px;
    height: 54px;
    margin: 0 auto 14px;
    border-radius: 8px;
    background: #16a34a;
    color: white;
    border: 3px solid #0f172a;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 30px;
    font-weight: 900;
  }

  .thank-you h2 {
    margin: 0 0 8px;
  }

  .thank-you p {
    margin: 0;
    color: #166534;
    font-weight: 700;
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
    font-weight: 900;
  }

  .back-link {
    display: block;
    margin-top: 24px;
    text-align: center;
    color: #2563eb;
    font-weight: 900;
    text-decoration: none;
  }

  .back-link:hover {
    text-decoration: underline;
  }
</style>