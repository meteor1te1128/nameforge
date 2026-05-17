/* ════════════════════════════════════
   NameCraft — Main JavaScript
   ════════════════════════════════════ */

// ── CONFIG ──────────────────────────
// Replace with your Anthropic API key.
// For production, route through a Cloudflare Worker to keep the key private.
const API_KEY = 'YOUR_ANTHROPIC_API_KEY';
const API_URL = 'https://api.anthropic.com/v1/messages';

// ── STATE ────────────────────────────
let namingType = 'baby';
let selectedGender = 'boy';
let selectedOrigin = 'any';
let isGenerating = false;

// ── TYPE TABS ────────────────────────
document.querySelectorAll('.tab').forEach(tab => {
  tab.addEventListener('click', () => {
    document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
    tab.classList.add('active');
    namingType = tab.dataset.type;
    adaptFormToType(namingType);
  });
});

function adaptFormToType(type) {
  const surnameRow = document.getElementById('row-surname');
  const genderField = document.getElementById('field-gender');
  const vibeInput = document.getElementById('vibe');
  const originChips = document.getElementById('originChips');

  if (type === 'baby') {
    surnameRow.style.display = 'grid';
    genderField.style.display = 'block';
    originChips.style.display = 'block';
    vibeInput.placeholder = 'e.g. strong & timeless, gentle & poetic, adventurous';
  } else if (type === 'pet') {
    surnameRow.style.display = 'grid';
    genderField.style.display = 'block';
    originChips.style.display = 'block';
    vibeInput.placeholder = 'e.g. playful & cute, majestic, quirky, loyal';
  } else if (type === 'brand') {
    surnameRow.style.display = 'none';
    originChips.style.display = 'none';
    vibeInput.placeholder = 'e.g. luxury skincare, approachable tech startup, artisan coffee';
  }
}

// ── GENDER SEGMENTED CONTROL ─────────
document.querySelectorAll('.seg').forEach(seg => {
  seg.addEventListener('click', () => {
    document.querySelectorAll('.seg').forEach(s => s.classList.remove('active'));
    seg.classList.add('active');
    selectedGender = seg.dataset.val;
  });
});

// ── ORIGIN CHIPS ─────────────────────
document.querySelectorAll('.chip').forEach(chip => {
  chip.addEventListener('click', () => {
    document.querySelectorAll('.chip').forEach(c => c.classList.remove('active'));
    chip.classList.add('active');
    selectedOrigin = chip.dataset.val;
  });
});
// Default: select 'Any'
document.querySelector('.chip[data-val="any"]').classList.add('active');

// ── GENERATE ─────────────────────────
async function generateNames() {
  if (isGenerating) return;

  const vibe = document.getElementById('vibe').value.trim();
  const avoid = document.getElementById('avoid').value.trim();
  const surname = document.getElementById('surname')?.value.trim() || '';

  if (!vibe) {
    shakeInput('vibe');
    showToast('Tell us the feeling you want the name to carry.');
    return;
  }

  setLoading(true);

  try {
    const prompt = buildPrompt({ type: namingType, gender: selectedGender, origin: selectedOrigin, vibe, avoid, surname });
    const names = await callAPI(prompt);
    renderResults(names);
  } catch (err) {
    console.error(err);
    showToast('Something went wrong — please try again in a moment.');
  } finally {
    setLoading(false);
  }
}

function buildPrompt({ type, gender, origin, vibe, avoid, surname }) {
  const typeContexts = {
    baby: `a baby's first name (${gender})${surname ? `, paired with the last name "${surname}"` : ''}`,
    pet: `a pet's name (${gender})`,
    brand: 'a brand name'
  };

  const originLine = (type !== 'brand' && origin !== 'any')
    ? `\n- Cultural/linguistic origin: ${origin}`
    : '';

  const avoidLine = avoid ? `\n- Avoid: ${avoid}` : '';

  return `You are an expert naming consultant with deep knowledge of etymology, linguistics, literature, and cultural heritage.

Generate exactly 3 names for ${typeContexts[type]}.

User preferences:
- Feeling/vibe: ${vibe}${originLine}${avoidLine}

Return ONLY a valid JSON array — no markdown, no commentary, nothing else:
[
  {
    "name": "The name",
    "pronunciation": "phonetic guide, e.g. ROW-an",
    "meaning": "2–3 sentences. Cover the etymology, cultural roots, and why it fits the requested vibe. Be genuinely interesting — mention literary references, historical figures, or linguistic origins where relevant."
  }
]

Rules:
- Each name must be meaningfully different in style (e.g. classic, poetic, modern)
- Names must be genuinely beautiful and feel intentional, not generic
- Pronunciation guide in plain English phonetics (no IPA)
- Meaning must be substantive and culturally informed — not just "it means strength"
- Output raw JSON only`;
}

async function callAPI(prompt) {
  const res = await fetch(API_URL, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-api-key': API_KEY,
      'anthropic-version': '2023-06-01',
      'anthropic-dangerous-direct-browser-access': 'true'
    },
    body: JSON.stringify({
      model: 'claude-sonnet-4-20250514',
      max_tokens: 1200,
      messages: [{ role: 'user', content: prompt }]
    })
  });

  if (!res.ok) {
    const body = await res.text();
    throw new Error(`API ${res.status}: ${body}`);
  }

  const data = await res.json();
  const raw = data.content[0]?.text || '';
  const clean = raw.replace(/```json|```/g, '').trim();
  return JSON.parse(clean);
}

// ── RENDER ────────────────────────────
function renderResults(names) {
  const empty = document.getElementById('resultsEmpty');
  const content = document.getElementById('resultsContent');
  const list = document.getElementById('namesList');

  empty.style.display = 'none';
  content.style.display = 'block';
  list.innerHTML = '';

  names.forEach((n, i) => {
    const card = document.createElement('div');
    card.className = 'name-card';
    card.style.animationDelay = `${i * 0.1}s`;
    card.innerHTML = `
      <div class="name-main">
        <span class="name-text">${escHtml(n.name)}</span>
        <span class="name-pron">${escHtml(n.pronunciation)}</span>
      </div>
      <p class="name-meaning">${escHtml(n.meaning)}</p>
    `;
    list.appendChild(card);
  });

  // Smooth scroll to results on mobile
  if (window.innerWidth < 900) {
    setTimeout(() => {
      document.getElementById('genResults').scrollIntoView({ behavior: 'smooth', block: 'start' });
    }, 100);
  }
}

// ── MODAL ─────────────────────────────
function openModal() {
  document.getElementById('modalBg').classList.add('open');
  document.body.style.overflow = 'hidden';
}

function closeModal() {
  document.getElementById('modalBg').classList.remove('open');
  document.body.style.overflow = '';
}

// Close modal on backdrop click
document.getElementById('modalBg').addEventListener('click', function(e) {
  if (e.target === this) closeModal();
});

// Close on Escape
document.addEventListener('keydown', e => {
  if (e.key === 'Escape') closeModal();
});

// ── HELPERS ───────────────────────────
function setLoading(state) {
  isGenerating = state;
  const btn = document.getElementById('genBtn');
  const text = document.getElementById('btnText');
  const load = document.getElementById('btnLoad');
  btn.disabled = state;
  text.style.display = state ? 'none' : '';
  load.style.display = state ? '' : 'none';
}

function shakeInput(id) {
  const el = document.getElementById(id);
  el.style.animation = 'none';
  el.offsetHeight; // reflow
  el.style.animation = 'shake .4s ease';
  el.focus();
}

function showToast(msg) {
  const existing = document.querySelector('.toast');
  if (existing) existing.remove();

  const toast = document.createElement('div');
  toast.className = 'toast';
  toast.style.cssText = `
    position: fixed;
    bottom: 28px;
    left: 50%;
    transform: translateX(-50%);
    background: #1c1a13;
    color: #f8f5ef;
    font-family: 'DM Sans', sans-serif;
    font-size: 14px;
    padding: 13px 22px;
    border-radius: 999px;
    z-index: 9999;
    box-shadow: 0 8px 32px rgba(0,0,0,.25);
    white-space: nowrap;
    animation: toastIn .25s ease;
  `;
  toast.textContent = msg;
  document.body.appendChild(toast);
  setTimeout(() => toast.style.opacity = '0', 2800);
  setTimeout(() => toast.remove(), 3200);
}

function escHtml(str) {
  return (str || '').replace(/&/g,'&amp;').replace(/</g,'&lt;').replace(/>/g,'&gt;').replace(/"/g,'&quot;');
}

// ── SCROLL REVEAL ─────────────────────
const revealObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.style.opacity = '1';
      entry.target.style.transform = 'translateY(0)';
      revealObserver.unobserve(entry.target);
    }
  });
}, { threshold: 0.08 });

document.querySelectorAll('.step-card, .review, .pricing-card').forEach(el => {
  el.style.opacity = '0';
  el.style.transform = 'translateY(20px)';
  el.style.transition = 'opacity .5s ease, transform .5s ease';
  revealObserver.observe(el);
});

// ── INJECT GLOBAL KEYFRAMES ───────────
const styleEl = document.createElement('style');
styleEl.textContent = `
  @keyframes shake {
    0%,100% { transform: translateX(0); }
    20%,60%  { transform: translateX(-5px); }
    40%,80%  { transform: translateX(5px); }
  }
  @keyframes toastIn {
    from { opacity: 0; transform: translateX(-50%) translateY(8px); }
    to   { opacity: 1; transform: translateX(-50%) translateY(0); }
  }
`;
document.head.appendChild(styleEl);
