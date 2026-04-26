<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>ScanSight — OCR PDF Converter</title>
<link href="https://fonts.googleapis.com/css2?family=Syne:wght@400;500;600;700;800&family=DM+Mono:ital,wght@0,300;0,400;0,500;1,300&family=Lato:wght@300;400;700&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #0a0a0f;
    --surface: #111118;
    --surface2: #16161f;
    --border: #2a2a3a;
    --accent: #00e5ff;
    --accent2: #7c3aed;
    --accent3: #f59e0b;
    --text: #e8e8f0;
    --text-dim: #8888aa;
    --text-dimmer: #44445a;
    --success: #10b981;
    --error: #ef4444;
    --radius: 16px;
    --radius-sm: 8px;
  }

  * { box-sizing: border-box; margin: 0; padding: 0; }

  body {
    font-family: 'Lato', sans-serif;
    background: var(--bg);
    color: var(--text);
    min-height: 100vh;
    overflow-x: hidden;
    position: relative;
  }

  /* Animated background */
  body::before {
    content: '';
    position: fixed;
    top: -50%;
    left: -50%;
    width: 200%;
    height: 200%;
    background: 
      radial-gradient(ellipse 600px 400px at 20% 20%, rgba(0,229,255,0.04) 0%, transparent 70%),
      radial-gradient(ellipse 500px 350px at 80% 70%, rgba(124,58,237,0.05) 0%, transparent 70%),
      radial-gradient(ellipse 400px 300px at 60% 10%, rgba(245,158,11,0.03) 0%, transparent 60%);
    pointer-events: none;
    z-index: 0;
    animation: bgShift 20s ease-in-out infinite alternate;
  }

  @keyframes bgShift {
    0% { transform: translate(0,0) rotate(0deg); }
    100% { transform: translate(2%,2%) rotate(1deg); }
  }

  /* Grid overlay */
  body::after {
    content: '';
    position: fixed;
    inset: 0;
    background-image: 
      linear-gradient(rgba(0,229,255,0.015) 1px, transparent 1px),
      linear-gradient(90deg, rgba(0,229,255,0.015) 1px, transparent 1px);
    background-size: 60px 60px;
    pointer-events: none;
    z-index: 0;
  }

  .container {
    position: relative;
    z-index: 1;
    max-width: 900px;
    margin: 0 auto;
    padding: 48px 24px 80px;
  }

  /* Header */
  header {
    text-align: center;
    margin-bottom: 60px;
    animation: fadeUp 0.7s ease both;
  }

  .logo {
    display: inline-flex;
    align-items: center;
    gap: 12px;
    margin-bottom: 28px;
  }

  .logo-icon {
    width: 44px;
    height: 44px;
    background: linear-gradient(135deg, var(--accent), var(--accent2));
    border-radius: 12px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 22px;
    box-shadow: 0 0 24px rgba(0,229,255,0.3);
  }

  .logo-text {
    font-family: 'Syne', sans-serif;
    font-size: 28px;
    font-weight: 800;
    background: linear-gradient(90deg, var(--accent), #a78bfa);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    letter-spacing: -0.5px;
  }

  h1 {
    font-family: 'Syne', sans-serif;
    font-size: clamp(36px, 6vw, 58px);
    font-weight: 800;
    line-height: 1.05;
    letter-spacing: -1.5px;
    margin-bottom: 16px;
  }

  h1 span {
    background: linear-gradient(90deg, var(--accent) 0%, #a78bfa 50%, var(--accent3) 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
  }

  .subtitle {
    font-size: 17px;
    color: var(--text-dim);
    font-weight: 300;
    line-height: 1.6;
    max-width: 520px;
    margin: 0 auto;
  }

  /* Badges */
  .badges {
    display: flex;
    gap: 10px;
    justify-content: center;
    flex-wrap: wrap;
    margin-top: 24px;
  }

  .badge {
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    font-weight: 500;
    padding: 5px 12px;
    border-radius: 100px;
    border: 1px solid;
    letter-spacing: 0.5px;
    text-transform: uppercase;
  }

  .badge-cyan { border-color: rgba(0,229,255,0.4); color: var(--accent); background: rgba(0,229,255,0.06); }
  .badge-purple { border-color: rgba(124,58,237,0.4); color: #a78bfa; background: rgba(124,58,237,0.06); }
  .badge-amber { border-color: rgba(245,158,11,0.4); color: var(--accent3); background: rgba(245,158,11,0.06); }

  /* Upload Zone */
  .upload-card {
    background: var(--surface);
    border: 1.5px solid var(--border);
    border-radius: var(--radius);
    padding: 40px;
    margin-bottom: 24px;
    animation: fadeUp 0.7s 0.1s ease both;
    transition: border-color 0.3s;
  }

  .drop-zone {
    border: 2px dashed var(--border);
    border-radius: var(--radius);
    padding: 52px 32px;
    text-align: center;
    cursor: pointer;
    transition: all 0.3s;
    position: relative;
    overflow: hidden;
    background: var(--surface2);
  }

  .drop-zone::before {
    content: '';
    position: absolute;
    inset: 0;
    background: radial-gradient(ellipse 80% 60% at 50% 50%, rgba(0,229,255,0.04), transparent);
    opacity: 0;
    transition: opacity 0.3s;
  }

  .drop-zone:hover, .drop-zone.drag-over {
    border-color: var(--accent);
    background: rgba(0,229,255,0.03);
  }

  .drop-zone:hover::before, .drop-zone.drag-over::before { opacity: 1; }
  .drop-zone.drag-over { transform: scale(1.01); }

  .drop-icon {
    width: 64px;
    height: 64px;
    margin: 0 auto 20px;
    background: linear-gradient(135deg, rgba(0,229,255,0.15), rgba(124,58,237,0.15));
    border-radius: 20px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 30px;
    border: 1px solid rgba(0,229,255,0.2);
    transition: transform 0.3s;
  }

  .drop-zone:hover .drop-icon { transform: translateY(-4px) scale(1.05); }

  .drop-title {
    font-family: 'Syne', sans-serif;
    font-size: 20px;
    font-weight: 700;
    margin-bottom: 8px;
  }

  .drop-sub {
    color: var(--text-dim);
    font-size: 14px;
    margin-bottom: 20px;
  }

  .browse-btn {
    display: inline-flex;
    align-items: center;
    gap: 8px;
    padding: 10px 24px;
    background: linear-gradient(135deg, rgba(0,229,255,0.15), rgba(124,58,237,0.15));
    border: 1px solid rgba(0,229,255,0.35);
    border-radius: 100px;
    color: var(--accent);
    font-size: 14px;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.25s;
    font-family: 'DM Mono', monospace;
    letter-spacing: 0.3px;
  }

  .browse-btn:hover {
    background: rgba(0,229,255,0.2);
    box-shadow: 0 0 20px rgba(0,229,255,0.2);
    transform: translateY(-1px);
  }

  #fileInput { display: none; }

  /* File preview */
  .file-preview {
    display: none;
    align-items: center;
    gap: 16px;
    padding: 16px 20px;
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: var(--radius-sm);
    margin-top: 20px;
  }

  .file-preview.show { display: flex; }

  .file-icon-box {
    width: 48px;
    height: 48px;
    background: linear-gradient(135deg, rgba(239,68,68,0.2), rgba(245,158,11,0.2));
    border-radius: 10px;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 22px;
    flex-shrink: 0;
    border: 1px solid rgba(239,68,68,0.25);
  }

  .file-info { flex: 1; min-width: 0; }
  .file-name {
    font-family: 'DM Mono', monospace;
    font-size: 13px;
    font-weight: 500;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    color: var(--text);
    margin-bottom: 4px;
  }
  .file-size { font-size: 12px; color: var(--text-dim); }

  .remove-file {
    background: none;
    border: none;
    color: var(--text-dimmer);
    cursor: pointer;
    font-size: 20px;
    padding: 4px;
    transition: color 0.2s;
    flex-shrink: 0;
  }
  .remove-file:hover { color: var(--error); }

  /* Settings */
  .settings-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 20px;
    margin-bottom: 24px;
    animation: fadeUp 0.7s 0.2s ease both;
  }

  @media (max-width: 600px) { .settings-grid { grid-template-columns: 1fr; } }

  .setting-card {
    background: var(--surface);
    border: 1.5px solid var(--border);
    border-radius: var(--radius);
    padding: 24px;
    transition: border-color 0.3s;
  }

  .setting-card:hover { border-color: rgba(0,229,255,0.25); }

  .setting-label {
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    font-weight: 500;
    color: var(--text-dim);
    text-transform: uppercase;
    letter-spacing: 1px;
    margin-bottom: 10px;
    display: flex;
    align-items: center;
    gap: 6px;
  }

  .setting-label span { font-size: 14px; }

  select, input[type="text"] {
    width: 100%;
    background: var(--surface2);
    border: 1px solid var(--border);
    border-radius: var(--radius-sm);
    padding: 11px 14px;
    color: var(--text);
    font-family: 'Lato', sans-serif;
    font-size: 14px;
    outline: none;
    transition: border-color 0.25s, box-shadow 0.25s;
    -webkit-appearance: none;
  }

  select { background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 12 12'%3E%3Cpath fill='%238888aa' d='M6 8L1 3h10z'/%3E%3C/svg%3E"); background-repeat: no-repeat; background-position: right 14px center; padding-right: 36px; cursor: pointer; }

  select:focus, input[type="text"]:focus {
    border-color: rgba(0,229,255,0.5);
    box-shadow: 0 0 0 3px rgba(0,229,255,0.08);
  }

  /* API Key note */
  .api-note {
    background: rgba(245,158,11,0.07);
    border: 1px solid rgba(245,158,11,0.2);
    border-radius: var(--radius-sm);
    padding: 14px 16px;
    font-size: 13px;
    color: #fcd34d;
    line-height: 1.6;
    margin-bottom: 24px;
    animation: fadeUp 0.7s 0.25s ease both;
  }

  .api-note strong { color: var(--accent3); }
  .api-note a { color: var(--accent3); text-decoration: underline; }

  /* Convert Button */
  .convert-btn {
    width: 100%;
    padding: 18px;
    background: linear-gradient(135deg, var(--accent) 0%, #7c3aed 100%);
    border: none;
    border-radius: var(--radius);
    color: #000;
    font-family: 'Syne', sans-serif;
    font-size: 17px;
    font-weight: 800;
    cursor: pointer;
    transition: all 0.3s;
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 10px;
    letter-spacing: -0.3px;
    position: relative;
    overflow: hidden;
    animation: fadeUp 0.7s 0.3s ease both;
  }

  .convert-btn::after {
    content: '';
    position: absolute;
    inset: 0;
    background: linear-gradient(135deg, rgba(255,255,255,0.15), transparent);
    opacity: 0;
    transition: opacity 0.3s;
  }

  .convert-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 12px 40px rgba(0,229,255,0.3), 0 4px 16px rgba(124,58,237,0.4);
  }

  .convert-btn:hover::after { opacity: 1; }
  .convert-btn:active { transform: translateY(0); }
  .convert-btn:disabled { opacity: 0.5; cursor: not-allowed; transform: none; box-shadow: none; }

  /* Progress */
  .progress-section {
    display: none;
    background: var(--surface);
    border: 1.5px solid var(--border);
    border-radius: var(--radius);
    padding: 32px;
    margin-top: 24px;
  }

  .progress-section.show { display: block; animation: fadeUp 0.4s ease both; }

  .progress-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 16px;
  }

  .progress-title {
    font-family: 'Syne', sans-serif;
    font-size: 15px;
    font-weight: 700;
  }

  .progress-pct {
    font-family: 'DM Mono', monospace;
    font-size: 13px;
    color: var(--accent);
    font-weight: 500;
  }

  .progress-bar-track {
    height: 6px;
    background: var(--surface2);
    border-radius: 100px;
    overflow: hidden;
    margin-bottom: 12px;
  }

  .progress-bar-fill {
    height: 100%;
    background: linear-gradient(90deg, var(--accent), var(--accent2));
    border-radius: 100px;
    width: 0%;
    transition: width 0.5s ease;
    box-shadow: 0 0 10px rgba(0,229,255,0.5);
  }

  .progress-log {
    font-family: 'DM Mono', monospace;
    font-size: 12px;
    color: var(--text-dim);
    height: 80px;
    overflow-y: auto;
    display: flex;
    flex-direction: column;
    gap: 4px;
  }

  .log-line { display: flex; gap: 8px; }
  .log-time { color: var(--text-dimmer); flex-shrink: 0; }
  .log-msg { color: var(--text-dim); }
  .log-msg.success { color: var(--success); }
  .log-msg.error { color: var(--error); }
  .log-msg.info { color: var(--accent); }

  /* Result */
  .result-section {
    display: none;
    background: linear-gradient(135deg, rgba(16,185,129,0.08), rgba(0,229,255,0.05));
    border: 1.5px solid rgba(16,185,129,0.3);
    border-radius: var(--radius);
    padding: 32px;
    margin-top: 24px;
    text-align: center;
  }

  .result-section.show { display: block; animation: fadeUp 0.4s ease both; }

  .result-icon { font-size: 52px; margin-bottom: 12px; }

  .result-title {
    font-family: 'Syne', sans-serif;
    font-size: 22px;
    font-weight: 800;
    color: var(--success);
    margin-bottom: 8px;
  }

  .result-sub { color: var(--text-dim); font-size: 14px; margin-bottom: 24px; }

  .download-btn {
    display: inline-flex;
    align-items: center;
    gap: 10px;
    padding: 14px 32px;
    background: linear-gradient(135deg, var(--success), #059669);
    border: none;
    border-radius: 100px;
    color: #fff;
    font-family: 'Syne', sans-serif;
    font-size: 15px;
    font-weight: 700;
    cursor: pointer;
    transition: all 0.3s;
    text-decoration: none;
  }

  .download-btn:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 24px rgba(16,185,129,0.4);
  }

  /* Error section */
  .error-section {
    display: none;
    background: rgba(239,68,68,0.07);
    border: 1.5px solid rgba(239,68,68,0.3);
    border-radius: var(--radius);
    padding: 24px;
    margin-top: 24px;
  }

  .error-section.show { display: block; animation: fadeUp 0.4s ease both; }
  .error-title { font-family: 'Syne', sans-serif; font-size: 16px; font-weight: 700; color: var(--error); margin-bottom: 8px; }
  .error-msg { font-size: 14px; color: #fca5a5; line-height: 1.6; font-family: 'DM Mono', monospace; }

  /* Features */
  .features {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 16px;
    margin-top: 48px;
    animation: fadeUp 0.7s 0.4s ease both;
  }

  @media (max-width: 600px) { .features { grid-template-columns: 1fr; } }

  .feature-card {
    background: var(--surface);
    border: 1.5px solid var(--border);
    border-radius: var(--radius);
    padding: 24px 20px;
    text-align: center;
    transition: all 0.3s;
  }

  .feature-card:hover {
    border-color: rgba(0,229,255,0.25);
    transform: translateY(-3px);
  }

  .feature-emoji { font-size: 28px; margin-bottom: 10px; }
  .feature-title { font-family: 'Syne', sans-serif; font-size: 14px; font-weight: 700; margin-bottom: 6px; }
  .feature-desc { font-size: 12px; color: var(--text-dim); line-height: 1.5; }

  /* How it works */
  .how-section {
    margin-top: 48px;
    animation: fadeUp 0.7s 0.5s ease both;
  }

  .section-title {
    font-family: 'Syne', sans-serif;
    font-size: 22px;
    font-weight: 800;
    letter-spacing: -0.5px;
    margin-bottom: 20px;
    color: var(--text-dim);
  }

  .steps {
    display: flex;
    gap: 0;
    position: relative;
  }

  .steps::before {
    content: '';
    position: absolute;
    top: 24px;
    left: 24px;
    right: 24px;
    height: 1px;
    background: linear-gradient(90deg, var(--accent), var(--accent2), transparent);
    opacity: 0.3;
  }

  .step {
    flex: 1;
    text-align: center;
    padding: 0 12px;
  }

  .step-num {
    width: 48px;
    height: 48px;
    margin: 0 auto 12px;
    background: var(--surface2);
    border: 1.5px solid var(--border);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-family: 'DM Mono', monospace;
    font-size: 16px;
    font-weight: 500;
    color: var(--accent);
    position: relative;
    z-index: 1;
    transition: all 0.3s;
  }

  .step:hover .step-num {
    border-color: var(--accent);
    background: rgba(0,229,255,0.1);
    box-shadow: 0 0 16px rgba(0,229,255,0.2);
  }

  .step-title { font-size: 13px; font-weight: 700; margin-bottom: 4px; font-family: 'Syne', sans-serif; }
  .step-desc { font-size: 12px; color: var(--text-dim); line-height: 1.5; }

  /* Animations */
  @keyframes fadeUp {
    from { opacity: 0; transform: translateY(20px); }
    to { opacity: 1; transform: translateY(0); }
  }

  @keyframes spin {
    to { transform: rotate(360deg); }
  }

  .spinning { animation: spin 1s linear infinite; display: inline-block; }

  /* Scrollbar */
  ::-webkit-scrollbar { width: 4px; }
  ::-webkit-scrollbar-track { background: var(--surface2); }
  ::-webkit-scrollbar-thumb { background: var(--border); border-radius: 2px; }

  /* Footer */
  footer {
    text-align: center;
    margin-top: 60px;
    font-size: 12px;
    color: var(--text-dimmer);
    font-family: 'DM Mono', monospace;
    animation: fadeUp 0.7s 0.6s ease both;
  }

  footer a { color: var(--text-dim); text-decoration: underline; }

  /* Chunk processing info */
  .chunk-info {
    display: none;
    font-family: 'DM Mono', monospace;
    font-size: 11px;
    color: var(--text-dim);
    text-align: center;
    margin-top: 8px;
    padding: 8px;
    background: var(--surface2);
    border-radius: var(--radius-sm);
    border: 1px solid var(--border);
  }
  .chunk-info.show { display: block; }
</style>
</head>
<body>
<div class="container">

  <header>
    <div class="logo">
      <div class="logo-icon">🔍</div>
      <span class="logo-text">ScanSight</span>
    </div>
    <h1>Make Any PDF<br><span>Searchable</span></h1>
    <p class="subtitle">Upload scanned or image-based PDFs and convert them into fully searchable, copy-able documents using AI-powered OCR.</p>
    <div class="badges">
      <span class="badge badge-cyan">⚡ All Languages</span>
      <span class="badge badge-purple">🔒 No Data Stored</span>
      <span class="badge badge-amber">📁 Up to 4 GB</span>
    </div>
  </header>

  <!-- API Key Notice -->
  <div class="api-note">
    <strong>🔑 Free API Required:</strong> This tool uses the <a href="https://ocr.space/ocrapi" target="_blank">OCR.space API</a> (free tier: 25k requests/month).
    Get your free API key at <a href="https://ocr.space/ocrapi/freekey" target="_blank">ocr.space/ocrapi/freekey</a> — it takes 30 seconds.
    Your key is never stored and stays in your browser only.
  </div>

  <!-- Upload Card -->
  <div class="upload-card">
    <div class="drop-zone" id="dropZone">
      <div class="drop-icon">📄</div>
      <div class="drop-title">Drop your PDF here</div>
      <div class="drop-sub">Drag & drop or click to browse — any size up to 4 GB</div>
      <label class="browse-btn" for="fileInput">
        📂 Browse Files
      </label>
      <input type="file" id="fileInput" accept=".pdf,application/pdf">
    </div>

    <div class="file-preview" id="filePreview">
      <div class="file-icon-box">📕</div>
      <div class="file-info">
        <div class="file-name" id="fileName">document.pdf</div>
        <div class="file-size" id="fileSize">—</div>
      </div>
      <button class="remove-file" id="removeFile" title="Remove file">✕</button>
    </div>
  </div>

  <!-- Settings -->
  <div class="settings-grid">
    <div class="setting-card">
      <div class="setting-label"><span>🌍</span> OCR Language</div>
      <select id="langSelect">
        <option value="eng">English</option>
        <option value="hin">Hindi (हिन्दी)</option>
        <option value="ara">Arabic (العربية)</option>
        <option value="chi_sim">Chinese Simplified</option>
        <option value="chi_tra">Chinese Traditional</option>
        <option value="jpn">Japanese (日本語)</option>
        <option value="kor">Korean (한국어)</option>
        <option value="fra">French (Français)</option>
        <option value="deu">German (Deutsch)</option>
        <option value="spa">Spanish (Español)</option>
        <option value="por">Portuguese (Português)</option>
        <option value="rus">Russian (Русский)</op
