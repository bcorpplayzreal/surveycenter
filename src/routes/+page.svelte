<script>
//Hello there MMT, you guys should def let me on the tech team :)
    import { onMount } from 'svelte';
    import { supabase } from '$lib/supabase';

  //data for login
    let username = $state("");
    let password = $state("");
    let user = $state(null);
    let message = $state("");
    let accountWarning = $state("");
  
    //survey data from Supabase
    let surveys = $state([]);
    let responsesBySurvey = $state({});
    let siteUrl = $state("");
  
    //new survey form fields
    let title = $state("");
    let question = $state("");
    let option1 = $state("");
    let option2 = $state("");
    let option3 = $state("");
    let option4 = $state("");
    //warnings
    let formWarning = $state("");
  
    //loads the currently lgged in user
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
    function clearAccountWarning() {
  accountWarning = "";
}
//it doesnt work unless i do this but i dont want emails in here anymore :/
function usernameToEmail(username) {
  return `${username.trim().toLowerCase()}@surveycenter.local`;
}

function isValidUsername(username) {
  return /^[a-zA-Z0-9_]{3,20}$/.test(username.trim());
}

function getDisplayName() {
  return user?.user_metadata?.username || user?.email;
}

//sign up feature woahh so cool ikr
async function signUp() {
  message = "";
  accountWarning = "";

  if (!username.trim() || !password.trim()) {
    accountWarning = "Please enter an username and a password to create an account.";
    return;
  }
  if (!isValidUsername(username)) {
    accountWarning = "Username must be 3-20 characters and can only use letters, numbers, and underscores.";
    return;
}
const fakeEmail = usernameToEmail(username);

const { data, error } = await supabase.auth.signUp({
  email: fakeEmail,
  password,
  options: {
    data: {
      username: username.trim()
    }
  }
});

if (error) {
  message = "Could not create account. Try a different username.";
  return;
}
  console.log("signUp data:", data);
  message = "Account created successfully.";
  await loadUser();
}
  
    async function signIn() {
      message = "";
      accountWarning = "";
      if (!username.trim() || !password.trim()) {
    accountWarning = "Please enter your username and password to sign in.";
    return;
  }
  const fakeEmail = usernameToEmail(username);

  const { data, error } = await supabase.auth.signInWithPassword({
    email: fakeEmail,
    password
  });
      if (error) {
        message = "Please check that your username and password is correct, then try again";
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
      responsesBySurvey = {};
      message = "Signed out.";
    }
  //loads surveys that are owned by the signed in user
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
      await loadAnalytics(data);
    }
  
    //loads the responses for each survey to calculated analytics
    async function loadAnalytics(surveyList) {
      let newResponsesBySurvey = {};
  
      for (const survey of surveyList) {
        const { data, error } = await supabase
          .from('survey_responses')
          .select('*')
          .eq('survey_id', survey.id);
  
        if (error) {
          console.log("loadAnalytics error:", error);
          message = error.message;
          continue;
        }
  
        newResponsesBySurvey[survey.id] = data;
      }
  
      responsesBySurvey = newResponsesBySurvey;
    }
  
    function getResponses(survey) {
      return responsesBySurvey[survey.id] || [];
    }
  
    function countAnswer(survey, answer) {
      return getResponses(survey).filter((response) => response.answer === answer).length;
    }

    function totalDashboardResponses() {
  return surveys.reduce((sum, survey) => {
    return sum + totalResponses(survey);
  }, 0);
}

function averageResponsesPerSurvey() {
  if (surveys.length === 0) {
    return 0;
  }

  return (totalDashboardResponses() / surveys.length).toFixed(1);
}

function mostActiveSurvey() {
  if (surveys.length === 0) {
    return "No surveys yet";
  }

  let topSurvey = surveys[0];
  let topCount = totalResponses(topSurvey);

  for (const survey of surveys) {
    const count = totalResponses(survey);

    if (count > topCount) {
      topSurvey = survey;
      topCount = count;
    }
  }

  if (topCount === 0) {
    return "No responses yet";
  }

  return `${topSurvey.title} (${topCount} response${topCount === 1 ? "" : "s"})`;
}
    function totalResponses(survey) {
      return getResponses(survey).length;
    }
    //calculates the percentage of responses from an answer choice
    function percentAnswer(survey, answer) {
      const total = totalResponses(survey);

  if (total === 0) {
    return 0;
  }

  return Math.round((countAnswer(survey, answer) / total) * 100);
}

//finds which choice has the most votes. 
function mostPopularAnswer(survey) {
  const options = [
    survey.option_1,
    survey.option_2,
    survey.option_3,
    survey.option_4
  ];

  let topAnswer = options[0];
  let topCount = countAnswer(survey, topAnswer);

  for (const option of options) {
    const count = countAnswer(survey, option);

    if (count > topCount) {
      topAnswer = option;
      topCount = count;
    }
  }

  if (topCount === 0) {
    return "No responses yet";
  }

  return `${topAnswer} is winning with ${topCount} vote${topCount === 1 ? "" : "s"} (${percentAnswer(survey, topAnswer)}%)`;
}

//creates new survey and saves to supabase
async function createSurvey() {
  message = "";
  formWarning = "";

  if (!title.trim() || !question.trim() || !option1.trim() || !option2.trim() || !option3.trim() || !option4.trim()) {
    formWarning = "Please fill out every box before creating your survey.";
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
  siteUrl = window.location.origin;
  loadUser();
});

function getSurveyUrl(survey) {
  return `${siteUrl}/survey/${survey.id}`;
}

async function copySurveyLink(survey) {
  const url = getSurveyUrl(survey);

  try {
    await navigator.clipboard.writeText(url);
    message = "Survey link copied!";
  } catch (error) {
    console.log("copy error:", error);
    message = "Could not copy. You can manually copy the link.";
  }
}
  </script>
  
  <main class="page">
    <div class="stars-layer" aria-hidden="true">
      <span class="star star-1">★</span>
      <span class="star star-2">✦</span>
      <span class="star star-3">★</span>
      <span class="star star-4">✧</span>
      <span class="star star-5">★</span>
      <span class="star star-6">✦</span>
    </div>
  
    <section class="card">
            <div class="header">
        <div class="logo">SC</div>
        <h1>★ SurveyCenter ★</h1>
        <p>Create and share simple surveys.</p>
      </div>
  
      {#if user}
        <div class="signed-in-box">
          <p>You are signed in as</p>
          <strong>{getDisplayName()}</strong>
            </div>
  <div class="summary-cards">
  <div class="summary-card">
    <span>Total Surveys</span>
    <strong>{surveys.length}</strong>
  </div>

  <div class="summary-card">
    <span>Total Responses</span>
    <strong>{totalDashboardResponses()}</strong>
  </div>

  <div class="summary-card">
    <span>Average Responses</span>
    <strong>{averageResponsesPerSurvey()}</strong>
  </div>

  <div class="summary-card wide">
    <span>Most Active Survey</span>
    <strong>{mostActiveSurvey()}</strong>
  </div>
</div>
        <div class="survey-form">
          <h2>Create a Survey</h2>
  
          <label>
            Survey title
            <input
              bind:value={title}
              placeholder="Mustang Math Recruitment Form"
            />
          </label>
  
          <label>
            Question
            <input
              bind:value={question}
              placeholder="What teams are you interested in"
            />
          </label>
  
          <label>
            Option 1
            <input bind:value={option1} placeholder="Tech" />
          </label>
  
          <label>
            Option 2
            <input bind:value={option2} placeholder="Problem Writing" />
          </label>
  
          <label>
            Option 3
            <input bind:value={option3} placeholder="Tournament Design" />
          </label>
  
          <label>
            Option 4
            <input bind:value={option4} placeholder="Curriculum Development" />
          </label>
  
          {#if formWarning}
          <p class="form-warning">{formWarning}</p>
        {/if}
        
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
  
                <div class="share-box">
                    <p class="share-label">Share this survey:</p>
                  
                    <input
                      class="share-input"
                      readonly
                      value={getSurveyUrl(survey)}
                    />
                  
                    <div class="share-buttons">
                      <a class="survey-link" href={`/survey/${survey.id}`}>
                        Open survey
                      </a>
                  
                      <button class="copy-button" onclick={() => copySurveyLink(survey)}>
                        Copy link
                      </button>
                    </div>
                  </div>
  
                <div class="analytics-box">
                  <h4>Analytics</h4>
  
                  <p class="total">
                    Total responses: {totalResponses(survey)}
                  </p>
                  <p class="top-answer">
                    Most popular: {mostPopularAnswer(survey)}
                  </p>
                  <div class="result-row">
                    <div class="result-top">
                      <span>{survey.option_1}</span>
                      <strong>
                        {countAnswer(survey, survey.option_1)} votes · {percentAnswer(survey, survey.option_1)}%
                      </strong>
                    </div>
                  
                    <div class="bar">
                      <div
                        class="bar-fill"
                        style={`width: ${percentAnswer(survey, survey.option_1)}%`}
                      ></div>
                    </div>
                  </div>
                  
                  <div class="result-row">
                    <div class="result-top">
                      <span>{survey.option_2}</span>
                      <strong>
                        {countAnswer(survey, survey.option_2)} votes · {percentAnswer(survey, survey.option_2)}%
                      </strong>
                    </div>
                  
                    <div class="bar">
                      <div
                        class="bar-fill"
                        style={`width: ${percentAnswer(survey, survey.option_2)}%`}
                      ></div>
                    </div>
                  </div>
                  
                  <div class="result-row">
                    <div class="result-top">
                      <span>{survey.option_3}</span>
                      <strong>
                        {countAnswer(survey, survey.option_3)} votes · {percentAnswer(survey, survey.option_3)}%
                      </strong>
                    </div>
                  
                    <div class="bar">
                      <div
                        class="bar-fill"
                        style={`width: ${percentAnswer(survey, survey.option_3)}%`}
                      ></div>
                    </div>
                  </div>
                  
                  <div class="result-row">
                    <div class="result-top">
                      <span>{survey.option_4}</span>
                      <strong>
                        {countAnswer(survey, survey.option_4)} votes · {percentAnswer(survey, survey.option_4)}%
                      </strong>
                    </div>
                  
                    <div class="bar">
                      <div
                        class="bar-fill"
                        style={`width: ${percentAnswer(survey, survey.option_4)}%`}
                      ></div>
                    </div>
                  </div>
                </div>
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
            Username
            <input
            bind:value={username}
            oninput={clearAccountWarning}
            type="text"
            placeholder="Mustang"
          />
          </label>
  
          <label>
            Password
            <input
            bind:value={password}
            oninput={clearAccountWarning}
            type="password"
            placeholder="Enter your password"
          />
          </label>
  
          <button class="primary-button" onclick={signIn}>
            Sign in
          </button>
          {#if accountWarning}
          <p class="account-warning">{accountWarning}</p>
        {/if}

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
  
    .survey-link {
      display: inline-block;
      margin-top: 12px;
      color: #2563eb;
      font-weight: bold;
      text-decoration: none;
    }
  
    .survey-link:hover {
      text-decoration: underline;
    }
  
    .analytics-box {
      margin-top: 16px;
      background: white;
      border: 1px solid #e2e8f0;
      border-radius: 14px;
      padding: 14px;
    }
  
    .analytics-box h4 {
      margin: 0 0 10px;
      font-size: 16px;
    }
  
    .total {
      margin: 0 0 12px;
      color: #64748b;
      font-size: 14px;
    }
  
    .result-row {
  background: #f1f5f9;
  border-radius: 12px;
  padding: 10px 12px;
  margin-bottom: 10px;
}

.result-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  margin-bottom: 8px;
}

.result-top span {
  color: #334155;
  font-weight: 600;
}

.result-top strong {
  color: #2563eb;
  font-size: 14px;
  white-space: nowrap;
}

.bar {
  width: 100%;
  height: 10px;
  background: #dbeafe;
  border-radius: 999px;
  overflow: hidden;
}

.bar-fill {
  height: 100%;
  background: #2563eb;
  border-radius: 999px;
}
  
    .message {
      margin-top: 18px;
      text-align: center;
      color: #2563eb;
      font-weight: 600;
    }
    .share-box {
  margin-top: 14px;
  background: #eff6ff;
  border: 1px solid #bfdbfe;
  border-radius: 14px;
  padding: 14px;
}

.share-label {
  margin: 0 0 8px;
  font-size: 14px;
  color: #475569;
  font-weight: 600;
}

.share-input {
  width: 100%;
  box-sizing: border-box;
  background: white;
  color: #334155;
  font-size: 14px;
  margin-bottom: 10px;
}

.share-buttons {
  display: flex;
  gap: 10px;
  align-items: center;
}

.copy-button {
  background: #2563eb;
  color: white;
  padding: 10px 12px;
  font-size: 14px;
}

.copy-button:hover {
  background: #1d4ed8;
}
.top-answer {
  background: #ecfdf5;
  border: 1px solid #bbf7d0;
  color: #166534;
  border-radius: 10px;
  padding: 10px 12px;
  font-size: 14px;
  font-weight: 700;
  margin: 0 0 12px;
}
.form-warning {
  background: #fef2f2;
  border: 1px solid #fecaca;
  color: #b91c1c;
  border-radius: 12px;
  padding: 12px 14px;
  font-size: 14px;
  font-weight: 700;
  margin: 0;
}
.summary-cards {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 12px;
  margin-bottom: 24px;
}

.summary-card {
  background: #eff6ff;
  border: 1px solid #bfdbfe;
  border-radius: 16px;
  padding: 16px;
}

.summary-card span {
  display: block;
  color: #475569;
  font-size: 13px;
  font-weight: 700;
  margin-bottom: 6px;
}

.summary-card strong {
  color: #2563eb;
  font-size: 24px;
}

.summary-card.wide {
  grid-column: span 2;
}

.summary-card.wide strong {
  display: block;
  font-size: 17px;
  line-height: 1.4;
}
.page {
  min-height: 100vh;
  display: flex;
  align-items: flex-start;
  justify-content: center;
  padding: 48px 28px;
  position: relative;
  overflow-x: hidden;
}

.card {
  width: 100%;
  max-width: 1100px;
  background: white;
  border: 3px solid #0f172a;
  border-radius: 10px;
  padding: 36px;
  box-shadow: 12px 12px 0 rgba(15, 23, 42, 0.18);
  position: relative;
  z-index: 2;
}

.logo {
  width: 64px;
  height: 64px;
  margin: 0 auto 16px;
  border-radius: 8px;
  background: #2563eb;
  color: white;
  border: 3px solid #0f172a;
  box-shadow: 5px 5px 0 #facc15;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 900;
  font-size: 22px;
}
.summary-cards {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 14px;
  margin-bottom: 28px;
}

.summary-card {
  background: #eff6ff;
  border: 3px solid #0f172a;
  border-radius: 8px;
  padding: 18px;
  box-shadow: 5px 5px 0 #bfdbfe;
}

.summary-card.wide {
  grid-column: span 2;
}

.survey-form,
.my-surveys,
.signed-in-box {
  border: 3px solid #0f172a;
  border-radius: 8px;
  padding: 22px;
  background: #ffffff;
  box-shadow: 6px 6px 0 #dbeafe;
}

.survey-form {
  margin-top: 24px;
}

.my-surveys {
  margin-top: 28px;
}

.survey-item {
  background: #f8fafc;
  border: 3px solid #0f172a;
  border-radius: 8px;
  padding: 18px;
  margin-bottom: 18px;
  box-shadow: 5px 5px 0 #e0f2fe;
}
input,
select {
  border: 3px solid #0f172a;
  border-radius: 6px;
}

button {
  border: 3px solid #0f172a;
  border-radius: 6px;
  box-shadow: 4px 4px 0 #0f172a;
}

button:hover {
  transform: translate(1px, 1px);
  box-shadow: 3px 3px 0 #0f172a;
}
.stars-layer {
  position: fixed;
  inset: 0;
  pointer-events: none;
  z-index: 1;
}

.star {
  position: absolute;
  color: #facc15;
  text-shadow: 2px 2px 0 #0f172a;
  opacity: 0.9;
}

.star-1 {
  top: 8%;
  left: 8%;
  font-size: 42px;
}

.star-2 {
  top: 16%;
  right: 10%;
  font-size: 34px;
}

.star-3 {
  top: 48%;
  left: 5%;
  font-size: 30px;
}

.star-4 {
  bottom: 18%;
  right: 7%;
  font-size: 46px;
}

.star-5 {
  bottom: 8%;
  left: 18%;
  font-size: 28px;
}

.star-6 {
  top: 68%;
  right: 18%;
  font-size: 26px;
}
.header h1 {
  letter-spacing: -1px;
}

@media (max-width: 800px) {
  .card {
    padding: 24px;
  }

  .summary-cards {
    grid-template-columns: 1fr 1fr;
  }

  .summary-card.wide {
    grid-column: span 2;
  }

  .star {
    opacity: 0.45;
  }

  
}
.account-warning {
  background: #fef2f2;
  border: 2px solid #ef4444;
  color: #991b1b;
  border-radius: 8px;
  padding: 12px 14px;
  font-size: 14px;
  font-weight: 800;
  margin: 0;
}
  </style>