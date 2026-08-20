---
layout: default
title: "Regular Pulse"
toc: false
---

<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,500;9..144,600&family=Source+Serif+4:opsz,wght@8..60,400;8..60,500&family=IBM+Plex+Sans:wght@400;500;600&display=swap" rel="stylesheet">

<div class="pulse-page">
  <header class="pulse-hero">
    <p class="pulse-kicker">Field notes</p>
    <h1 class="pulse-title">Regular Pulse</h1>
    <p class="pulse-lede">A running journal of what I’m building, reading, and figuring out—short notes, kept honest.</p>
  </header>

  <div class="pulse-rail" aria-hidden="true"></div>

  <ol class="pulse-feed">
    <li class="pulse-entry" style="--i: 0">
      <time class="pulse-date" datetime="2025-01-15">Jan 15, 2025</time>
      <p class="pulse-text">Working on improving the reinforcement learning model for delivery optimization. Spent the day analyzing feature importance and tuning hyperparameters.</p>
    </li>

    <li class="pulse-entry" style="--i: 1">
      <time class="pulse-date" datetime="2025-01-14">Jan 14, 2025</time>
      <p class="pulse-text">Reviewed the latest research on time-series foundation models. Started drafting ideas for a new project combining forecasting with uncertainty quantification.</p>
    </li>

    <li class="pulse-entry" style="--i: 2">
      <time class="pulse-date" datetime="2025-01-13">Jan 13, 2025</time>
      <p class="pulse-text">Attended a virtual workshop on MLOps best practices. Learned about new tools for model versioning and experiment tracking.</p>
    </li>

    <li class="pulse-entry" style="--i: 3">
      <time class="pulse-date" datetime="2025-01-12">Jan 12, 2025</time>
      <p class="pulse-text">Fixed a bug in the production forecasting pipeline. The issue was related to timezone handling in the data preprocessing step.</p>
    </li>

    <li class="pulse-entry" style="--i: 4">
      <time class="pulse-date" datetime="2025-01-11">Jan 11, 2025</time>
      <p class="pulse-text">Read a fascinating paper on diversity metrics in generative models. Thinking about how to apply similar concepts to our recommendation systems.</p>
    </li>
  </ol>
</div>

<style>
.pulse-page {
  --pulse-ink: #161616;
  --pulse-mute: #5a5a5a;
  --pulse-line: #d8dde3;
  --pulse-wash: #f2f5f8;
  --pulse-accent: #ff0f00;
  --pulse-serif: "Source Serif 4", "Times New Roman", Times, serif;
  --pulse-display: "Fraunces", "Times New Roman", Times, serif;
  --pulse-sans: "IBM Plex Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
  position: relative;
  margin: 0.5rem 0 2.5rem;
  color: var(--pulse-ink);
}

.pulse-page::before {
  content: "";
  position: absolute;
  inset: -1.25rem -1.5rem auto;
  height: 11rem;
  background:
    radial-gradient(ellipse 75% 65% at 8% 15%, rgba(255, 15, 0, 0.06), transparent 55%),
    linear-gradient(180deg, var(--pulse-wash) 0%, transparent 100%);
  pointer-events: none;
  z-index: 0;
  border-radius: 0;
}

.pulse-hero,
.pulse-feed {
  position: relative;
  z-index: 1;
}

.pulse-hero {
  margin-bottom: 2.25rem;
  padding-bottom: 1.5rem;
  border-bottom: 1px solid var(--pulse-line);
  animation: pulse-rise 0.7s ease-out both;
}

.pulse-kicker {
  margin: 0 0 0.55rem;
  font-family: var(--pulse-sans);
  font-size: 0.72rem;
  font-weight: 600;
  letter-spacing: 0.14em;
  text-transform: uppercase;
  color: var(--pulse-accent);
}

.pulse-title {
  margin: 0 0 0.7rem;
  font-family: var(--pulse-display);
  font-size: clamp(2.1rem, 5vw, 2.75rem);
  font-weight: 600;
  font-optical-sizing: auto;
  line-height: 1.1;
  letter-spacing: -0.02em;
  color: var(--pulse-ink);
}

.pulse-lede {
  margin: 0;
  max-width: 34rem;
  font-family: var(--pulse-serif);
  font-size: 1.1rem;
  font-weight: 400;
  line-height: 1.55;
  color: var(--pulse-mute);
}

.pulse-feed {
  list-style: none;
  margin: 0;
  padding: 0 0 0 1.35rem;
  border-left: 1px solid var(--pulse-line);
}

.pulse-entry {
  position: relative;
  margin: 0 0 1.85rem;
  padding: 0 0 0 1.15rem;
  animation: pulse-rise 0.65s ease-out both;
  animation-delay: calc(0.08s * var(--i, 0) + 0.12s);
}

.pulse-entry:last-child {
  margin-bottom: 0;
}

.pulse-entry::before {
  content: "";
  position: absolute;
  left: -1.35rem;
  top: 0.45rem;
  width: 0.55rem;
  height: 0.55rem;
  margin-left: -0.3rem;
  border-radius: 50%;
  background: var(--pulse-accent);
  box-shadow: 0 0 0 3px #fff;
  transition: transform 0.25s ease, box-shadow 0.25s ease;
}

.pulse-entry:hover::before {
  transform: scale(1.2);
  box-shadow: 0 0 0 4px rgba(255, 15, 0, 0.12);
}

.pulse-date {
  display: block;
  margin: 0 0 0.4rem;
  font-family: var(--pulse-sans);
  font-size: 0.78rem;
  font-weight: 600;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: var(--pulse-mute);
  transition: color 0.2s ease;
}

.pulse-entry:hover .pulse-date {
  color: var(--pulse-accent);
}

.pulse-text {
  margin: 0;
  font-family: var(--pulse-serif);
  font-size: 1.05rem;
  font-optical-sizing: auto;
  line-height: 1.7;
  color: var(--pulse-ink);
}

.pulse-text a {
  color: var(--pulse-accent);
  text-decoration: none;
  border-bottom: 1px solid rgba(255, 15, 0, 0.35);
  transition: border-color 0.2s ease;
}

.pulse-text a:hover {
  border-bottom-color: var(--pulse-accent);
}

.pulse-text code {
  font-family: "Inconsolata", "Menlo", monospace;
  font-size: 0.9em;
  padding: 0.1em 0.35em;
  background: var(--pulse-wash);
  color: #333;
}

.pulse-rail {
  display: none;
}

@keyframes pulse-rise {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (prefers-reduced-motion: reduce) {
  .pulse-hero,
  .pulse-entry {
    animation: none;
  }

  .pulse-entry::before {
    transition: none;
  }
}

@media (max-width: 600px) {
  .pulse-page::before {
    inset: -0.75rem -0.75rem auto;
    height: 9rem;
  }

  .pulse-hero {
    margin-bottom: 1.75rem;
    padding-bottom: 1.15rem;
  }

  .pulse-feed {
    padding-left: 1rem;
  }

  .pulse-entry {
    margin-bottom: 1.5rem;
    padding-left: 0.9rem;
  }

  .pulse-entry::before {
    left: -1rem;
    margin-left: -0.28rem;
  }

  .pulse-text {
    font-size: 1rem;
    line-height: 1.65;
  }
}
</style>
