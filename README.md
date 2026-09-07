# lineRed
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Redline — Read the fine print before you sign it</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link href="https://fonts.googleapis.com/css2?family=Newsreader:ital,wght@0,400;0,500;0,600;1,400&family=Public+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/mammoth/1.6.0/mammoth.browser.min.js"></script>
<style>
  :root{
    --ink:#1A1A17; --ink-soft:#4A473F; --paper:#FFFFFF; --paper-dim:#FAFAF8;
    --line:#DEDBD0; --redline:#B3312C; --redline-bg:#FBEBE9;
    --amber:#A8762E; --amber-bg:#FAF1E3; --sage:#54683F; --sage-bg:#EEF2E8;
  }
  *{box-sizing:border-box;}
  body{margin:0;background:var(--paper-dim);color:var(--ink);font-family:'Public Sans',sans-serif;-webkit-font-smoothing:antialiased;}
  .app{max-width:1180px;margin:0 auto;padding:0 24px 64px;}
  header{padding:30px 0 0;display:flex;align-items:baseline;justify-content:space-between;flex-wrap:wrap;gap:8px;}
  .logo{font-family:'Newsreader',serif;font-size:26px;font-weight:600;letter-spacing:-0.01em;}
  .logo em{font-style:italic;color:var(--redline);}
  .tagline{font-size:14px;color:var(--ink-soft);}
  nav{display:flex;gap:22px;border-bottom:1px solid var(--line);margin-top:22px;padding-bottom:0;}
  nav button{background:none;border:none;font-family:'Public Sans',sans-serif;font-size:13.5px;font-weight:600;color:var(--ink-soft);cursor:pointer;padding:10px 2px;border-bottom:2px solid transparent;}
  nav button.active{color:var(--ink);border-bottom-color:var(--redline);}
  .view{display:none;}
  .view.active{display:block;}

  .intro{max-width:640px;margin:56px auto 0;}
  .intro h1{font-family:'Newsreader',serif;font-weight:500;font-size:30px;line-height:1.25;margin:0 0 12px;}
  .intro p{font-size:15px;color:var(--ink-soft);line-height:1.6;margin:0 0 28px;}

  .field-row{display:flex;gap:14px;margin-bottom:16px;flex-wrap:wrap;}
  .field{flex:1;min-width:180px;}
  .field label{display:block;font-size:12px;font-weight:600;color:var(--ink-soft);margin-bottom:6px;}
  select, input[type=text], input[type=number]{
    width:100%;padding:10px 12px;border:1px solid var(--line);background:var(--paper);
    font-family:'Public Sans',sans-serif;font-size:14px;color:var(--ink);outline:none;
  }
  select:focus, input:focus{border-color:var(--ink-soft);}

  .prefs-toggle{font-size:13px;color:var(--redline);text-decoration:underline;text-underline-offset:3px;cursor:pointer;display:inline-block;margin-bottom:20px;}
  .prefs-panel{display:none;border:1px solid var(--line);background:var(--paper);padding:18px;margin-bottom:20px;}
  .prefs-panel.show{display:block;}
  .prefs-panel .note{font-size:12.5px;color:var(--ink-soft);margin-top:4px;}

  textarea{width:100%;min-height:260px;padding:20px;border:1px solid var(--line);background:var(--paper);
    font-family:'Newsreader',serif;font-size:16px;line-height:1.65;color:var(--ink);resize:vertical;outline:none;}
  textarea:focus{border-color:var(--ink-soft);}
  textarea::placeholder{color:#A8A499;}

  .controls{display:flex;align-items:center;justify-content:space-between;margin-top:16px;flex-wrap:wrap;gap:12px;}
  .char-count{font-size:13px;color:#A8A499;}
  button.primary{background:var(--ink);color:var(--paper);border:none;padding:13px 24px;font-family:'Public Sans',sans-serif;font-size:14px;font-weight:600;cursor:pointer;transition:background .15s ease;}
  button.primary:hover{background:#33312A;}
  button.primary:disabled{background:#B8B4A8;cursor:not-allowed;}
  button.secondary{background:var(--paper);color:var(--ink);border:1px solid var(--line);padding:8px 14px;font-family:'Public Sans',sans-serif;font-size:12.5px;font-weight:600;cursor:pointer;}
  button.secondary:hover{border-color:var(--ink-soft);}
  button.secondary:disabled{color:#B8B4A8;cursor:not-allowed;}

  .loading{max-width:620px;margin:100px auto;text-align:center;font-family:'Newsreader',serif;font-style:italic;font-size:19px;color:var(--ink-soft);}
  .loading .dot{animation:pulse 1.4s infinite ease-in-out;}
  .loading .dot:nth-child(2){animation-delay:.2s;} .loading .dot:nth-child(3){animation-delay:.4s;}
  @keyframes pulse{0%,80%,100%{opacity:.25;}40%{opacity:1;}}

  .error-box{max-width:620px;margin:40px auto;padding:20px 22px;background:var(--redline-bg);border:1px solid #E4C4C1;font-size:14px;line-height:1.6;}

  .risk-summary{display:flex;gap:36px;align-items:flex-start;flex-wrap:wrap;padding:24px 0;border-bottom:1px solid var(--line);margin-top:28px;}
  .risk-score-block{text-align:left;}
  .risk-score-num{font-family:'Newsreader',serif;font-size:52px;font-weight:600;line-height:1;}
  .risk-score-num.high{color:var(--redline);} .risk-score-num.medium{color:var(--amber);} .risk-score-num.low{color:var(--sage);}
  .risk-score-label{font-size:12.5px;color:var(--ink-soft);margin-top:4px;}
  .category-bars{flex:1;min-width:260px;display:grid;grid-template-columns:1fr 1fr;gap:10px 24px;}
  .cat-bar-row{font-size:12.5px;}
  .cat-bar-label{display:flex;justify-content:space-between;margin-bottom:4px;color:var(--ink-soft);}
  .cat-bar-track{height:5px;background:var(--line);}
  .cat-bar-fill{height:100%;}
  .cat-bar-fill.high{background:var(--redline);} .cat-bar-fill.medium{background:var(--amber);} .cat-bar-fill.low{background:var(--sage);}

  .missing-box{margin:22px 0;padding:16px 18px;background:var(--amber-bg);border:1px solid #E9D5AE;font-size:13.5px;line-height:1.7;}
  .missing-box strong{display:block;margin-bottom:6px;font-size:13px;}

  .results{display:none;grid-template-columns:1.5fr 1fr;gap:0;margin-top:8px;border:1px solid var(--line);}
  .results.show{display:grid;}
  @media (max-width:820px){.results.show{grid-template-columns:1fr;}}
  .doc-pane{padding:32px 36px;background:var(--paper);border-right:1px solid var(--line);font-family:'Newsreader',serif;font-size:16px;line-height:1.75;white-space:pre-wrap;max-height:760px;overflow-y:auto;}
  @media (max-width:820px){.doc-pane{border-right:none;border-bottom:1px solid var(--line);max-height:380px;}}
  mark{background:transparent;color:inherit;border-bottom:2px solid var(--redline);padding-bottom:1px;cursor:pointer;transition:background-color .15s ease;}
  mark.severity-medium{border-bottom-color:var(--amber);} mark.severity-low{border-bottom-color:var(--sage);}
  mark.active{background-color:var(--redline-bg);} mark.active.severity-medium{background-color:var(--amber-bg);} mark.active.severity-low{background-color:var(--sage-bg);}

  .findings-pane{background:var(--paper-dim);padding:24px 24px 16px;max-height:760px;overflow-y:auto;}
  .finding{padding:16px 0;border-bottom:1px solid var(--line);}
  .finding:last-child{border-bottom:none;}
  .finding-top{display:flex;align-items:center;gap:8px;margin-bottom:8px;cursor:pointer;}
  .severity-tag{font-size:11px;font-weight:700;padding:2px 8px;border:1px solid transparent;}
  .severity-tag.high{color:var(--redline);border-color:var(--redline);}
  .severity-tag.medium{color:var(--amber);border-color:var(--amber);}
  .severity-tag.low{color:var(--sage);border-color:var(--sage);}
  .finding-category{font-size:13px;font-weight:600;color:var(--ink);}
  .finding-quote{font-family:'Newsreader',serif;font-style:italic;font-size:14px;color:var(--ink-soft);margin:0 0 8px;line-height:1.5;cursor:pointer;}
  .finding-issue{font-size:13.5px;line-height:1.55;margin:0 0 8px;}
  .finding-fix{font-size:13px;line-height:1.5;color:var(--ink-soft);padding-left:12px;border-left:2px solid var(--line);margin-bottom:10px;}
  .finding-fix b{color:var(--ink);font-weight:600;}
  .finding-actions{display:flex;gap:8px;margin-bottom:8px;}
  .rewrite-box{font-size:13px;line-height:1.6;padding:10px 12px;background:var(--sage-bg);border:1px solid #C9D6BE;margin-top:6px;}
  .rewrite-box b{display:block;font-size:11px;text-transform:none;color:var(--sage);margin-bottom:4px;font-weight:700;}

  .all-clear{text-align:center;padding:40px 20px;font-family:'Newsreader',serif;font-style:italic;color:var(--sage);}

  .negotiation-block{margin-top:24px;padding-top:20px;border-top:1px solid var(--line);}
  .negotiation-output{margin-top:14px;padding:16px 18px;background:var(--paper);border:1px solid var(--line);font-size:13.5px;line-height:1.65;white-space:pre-wrap;display:none;}
  .negotiation-output.show{display:block;}

  .save-row{margin-top:20px;padding-top:16px;border-top:1px solid var(--line);display:flex;align-items:center;gap:12px;flex-wrap:wrap;}
  .save-msg{font-size:12.5px;color:var(--sage);display:none;}
  .save-msg.show{display:inline;}

  /* Library */
  .lib-item{padding:18px 20px;border:1px solid var(--line);background:var(--paper);margin-bottom:12px;cursor:pointer;}
  .lib-item:hover{border-color:var(--ink-soft);}
  .lib-top{display:flex;justify-content:space-between;align-items:baseline;gap:12px;flex-wrap:wrap;}
  .lib-title{font-family:'Newsreader',serif;font-size:17px;}
  .lib-date{font-size:12px;color:var(--ink-soft);}
  .lib-score{font-size:13px;font-weight:700;margin-top:6px;}
  .lib-score.high{color:var(--redline);} .lib-score.medium{color:var(--amber);} .lib-score.low{color:var(--sage);}
  .lib-empty{font-family:'Newsreader',serif;font-style:italic;color:var(--ink-soft);padding:60px 0;text-align:center;}
  .lib-detail{margin-top:20px;}
  .back-link{font-size:13px;color:var(--ink-soft);text-decoration:underline;cursor:pointer;margin-bottom:16px;display:inline-block;}

  /* Preferences */
  .prefs-form{max-width:520px;margin-top:24px;}
  .prefs-saved-msg{font-size:13px;color:var(--sage);margin-top:12px;display:none;}
  .prefs-saved-msg.show{display:block;}

  /* Pricing */
  .pricing{margin-top:56px;padding-top:32px;border-top:1px solid var(--line);}
  .pricing-note{font-size:12px;color:#A8A499;margin-bottom:20px;font-style:italic;}
  .pricing-grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(180px,1fr));gap:0;border:1px solid var(--line);}
  .plan{padding:22px 20px;border-right:1px solid var(--line);}
  .plan:last-child{border-right:none;}
  .plan-name{font-family:'Newsreader',serif;font-size:17px;margin-bottom:4px;}
  .plan-price{font-size:22px;font-weight:700;margin-bottom:12px;}
  .plan ul{margin:0;padding-left:16px;font-size:12.5px;line-height:1.8;color:var(--ink-soft);}

  footer{margin-top:28px;font-size:12.5px;color:#A8A499;line-height:1.6;}
  .reset-link{font-size:13px;color:var(--ink-soft);text-decoration:underline;text-underline-offset:3px;cursor:pointer;background:none;border:none;font-family:inherit;padding:0;display:none;}
  .reset-link.show{display:inline;}
</style>
</head>
<body>
<div class="app">
  <header>
    <div>
      <div class="logo">Red<em>line</em></div>
      <div class="tagline">Know what you're agreeing to before you sign</div>
    </div>
    <button class="reset-link" id="resetBtn">Start over</button>
  </header>
  <nav>
    <button class="nav-btn active" data-view="scan">New scan</button>
    <button class="nav-btn" data-view="library">My library</button>
    <button class="nav-btn" data-view="preferences">My preferred terms</button>
  </nav>

  <!-- SCAN VIEW -->
  <div class="view active" id="view-scan">
    <div id="idleScreen">
      <div class="intro">
        <h1>Paste a contract. We'll mark up what's worth a second look.</h1>
        <p>Freelance agreements, client contracts, NDAs — drop the text in below. Redline flags one-sided terms, vague scope, and clauses that usually favor the other side, with plain-English notes on each.</p>
      </div>

      <div class="field-row">
        <div class="field">
          <label for="roleSelect">Your role</label>
          <select id="roleSelect">
            <option value="general">General / not sure</option>
            <option value="Graphic Designer">Graphic designer</option>
            <option value="Web Developer">Web developer</option>
            <option value="App Developer">App developer</option>
            <option value="Photographer">Photographer</option>
            <option value="Videographer">Videographer</option>
            <option value="Copywriter">Copywriter</option>
            <option value="Social Media Manager">Social media manager</option>
            <option value="Software Developer">Software developer</option>
            <option value="Consultant">Consultant</option>
            <option value="Small Agency">Small agency</option>
            <option value="Content Creator">Content creator</option>
          </select>
        </div>
        <div class="field">
          <label for="jurisSelect">Jurisdiction</label>
          <select id="jurisSelect">
            <option value="not sure">Not sure / general</option>
            <option value="United States">United States</option>
            <option value="United Kingdom">United Kingdom</option>
            <option value="Ghana">Ghana</option>
            <option value="Nigeria">Nigeria</option>
            <option value="Canada">Canada</option>
          </select>
        </div>
      </div>

      <span class="prefs-toggle" id="prefsToggle">+ Check against my preferred terms</span>
      <div class="prefs-panel" id="prefsPanel">
        <div class="note" id="prefsPanelNote">You haven't set preferred terms yet.</div>
        <div class="note" id="prefsPanelSummary" style="display:none;"></div>
      </div>

      <textarea id="contractInput" placeholder="Paste your contract or agreement text here…"></textarea>
      <div class="controls">
        <span class="char-count" id="charCount">0 characters</span>
        <div style="display:flex;align-items:center;gap:14px;">
          <label for="fileInput" style="font-size:13px;color:var(--ink-soft);text-decoration:underline;cursor:pointer;">or upload a PDF/Word file</label>
          <input type="file" id="fileInput" accept=".pdf,.docx" style="display:none;">
          <button class="primary" id="checkBtn">Find the red flags</button>
        </div>
      </div>
      <div class="note" id="fileStatus" style="margin-top:8px;font-size:12.5px;color:var(--ink-soft);"></div>

      <div class="pricing">
        <div class="pricing-note">Reference pricing — plans below aren't wired up to real payment yet.</div>
        <div class="pricing-grid">
          <div class="plan">
            <div class="plan-name">Free</div>
            <div class="plan-price">$0</div>
            <ul><li>2 scans / month</li><li>Risk score</li><li>Basic red flags</li></ul>
          </div>
          <div class="plan">
            <div class="plan-name">Pro</div>
            <div class="plan-price">$15/mo</div>
            <ul><li>Unlimited scans</li><li>Preferred-terms comparison</li><li>Clause rewriting</li><li>Negotiation drafts</li><li>Contract library</li></ul>
          </div>
          <div class="plan">
            <div class="plan-name">Agency</div>
            <div class="plan-price">$29/mo</div>
            <ul><li>Everything in Pro</li><li>Team members</li><li>Shared clause library</li></ul>
          </div>
        </div>
      </div>
    </div>

    <div class="loading" id="loadingScreen" style="display:none;">Reading the fine print<span class="dot">.</span><span class="dot">.</span><span class="dot">.</span></div>
    <div class="error-box" id="errorBox" style="display:none;"></div>

    <div id="resultsWrap" style="display:none;">
      <div class="risk-summary" id="riskSummary"></div>
      <div class="missing-box" id="missingBox" style="display:none;"></div>
      <div class="results" id="resultsScreen">
        <div class="doc-pane" id="docPane"></div>
        <div class="findings-pane" id="findingsPane"></div>
      </div>
      <div class="negotiation-block">
        <button class="secondary" id="negotiateBtn">Draft a negotiation email</button>
        <div class="negotiation-output" id="negotiationOutput"></div>
      </div>
      <div class="save-row">
        <button class="secondary" id="saveBtn">Save to my library</button>
        <span class="save-msg" id="saveMsg">Saved.</span>
      </div>
    </div>

    <footer>Redline gives a quick first read, not legal advice — for anything high-stakes, have a lawyer look too.</footer>
  </div>

  <!-- LIBRARY VIEW -->
  <div class="view" id="view-library">
    <div class="intro" style="margin-top:32px;">
      <h1>My library</h1>
      <p>Contracts you've scanned and saved, stored on this device.</p>
    </div>
    <div id="libraryList"></div>
    <div id="libraryDetail" class="lib-detail" style="display:none;"></div>
  </div>

  <!-- PREFERENCES VIEW -->
  <div class="view" id="view-preferences">
    <div class="intro" style="margin-top:32px;">
      <h1>My preferred terms</h1>
      <p>Set your standard terms once. Every scan can check a contract against these and flag where it falls short.</p>
    </div>
    <div class="prefs-form">
      <div class="field-row">
        <div class="field"><label>Deposit required (%)</label><input type="number" id="prefDeposit" placeholder="e.g. 50"></div>
        <div class="field"><label>Payment terms</label><input type="text" id="prefPayment" placeholder="e.g. Net 14"></div>
      </div>
      <div class="field-row">
        <div class="field"><label>Max revision rounds</label><input type="number" id="prefRevisions" placeholder="e.g. 3"></div>
        <div class="field"><label>Cancellation / kill fee (%)</label><input type="number" id="prefKillFee" placeholder="e.g. 50"></div>
      </div>
      <div class="field-row">
        <div class="field"><label>Liability cap</label><input type="text" id="prefLiability" placeholder="e.g. capped at project value"></div>
        <div class="field"><label>IP transfers</label><input type="text" id="prefIP" placeholder="e.g. after full payment"></div>
      </div>
      <button class="primary" id="savePrefsBtn">Save my terms</button>
      <div class="prefs-saved-msg" id="prefsSavedMsg">Saved — this will be used in your next scan.</div>
    </div>
  </div>
</div>

<script>
const $ = id => document.getElementById(id);

// ---------- Nav ----------
document.querySelectorAll('.nav-btn').forEach(btn => {
  btn.addEventListener('click', () => {
    document.querySelectorAll('.nav-btn').forEach(b => b.classList.remove('active'));
    document.querySelectorAll('.view').forEach(v => v.classList.remove('active'));
    btn.classList.add('active');
    $('view-' + btn.dataset.view).classList.add('active');
    if (btn.dataset.view === 'library') loadLibrary();
    if (btn.dataset.view === 'preferences') loadPreferencesForm();
  });
});

$('resetBtn').addEventListener('click', () => {
  $('idleScreen').style.display = 'block';
  $('resultsWrap').style.display = 'none';
  $('errorBox').style.display = 'none';
  $('resetBtn').classList.remove('show');
  $('contractInput').value = '';
  $('charCount').textContent = '0 characters';
});

$('contractInput').addEventListener('input', () => {
  $('charCount').textContent = $('contractInput').value.length + ' characters';
});

if (window.pdfjsLib) {
  pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/3.11.174/pdf.worker.min.js';
}

$('fileInput').addEventListener('change', async (e) => {
  const file = e.target.files[0];
  if (!file) return;
  $('fileStatus').textContent = 'Reading ' + file.name + '…';
  try {
    let text = '';
    if (file.name.toLowerCase().endsWith('.pdf')) {
      const buf = await file.arrayBuffer();
      const pdf = await pdfjsLib.getDocument({ data: buf }).promise;
      for (let i = 1; i <= pdf.numPages; i++) {
        const page = await pdf.getPage(i);
        const content = await page.getTextContent();
        text += content.items.map(it => it.str).join(' ') + '\n\n';
      }
    } else if (file.name.toLowerCase().endsWith('.docx')) {
      const buf = await file.arrayBuffer();
      const result = await mammoth.extractRawText({ arrayBuffer: buf