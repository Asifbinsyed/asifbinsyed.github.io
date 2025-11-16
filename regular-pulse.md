---
layout: default
title: "Regular Pulse"
toc: false
---

# Regular Pulse

A daily journal of what I'm working on, learning, and thinking about.

<div class="pulse-content">
  <div class="pulse-entry">
    <div class="pulse-date">Jan 15, 2025</div>
    <div class="pulse-text">Working on improving the reinforcement learning model for delivery optimization. Spent the day analyzing feature importance and tuning hyperparameters.</div>
  </div>
  
  <div class="pulse-entry">
    <div class="pulse-date">Jan 14, 2025</div>
    <div class="pulse-text">Reviewed the latest research on time-series foundation models. Started drafting ideas for a new project combining forecasting with uncertainty quantification.</div>
  </div>
  
  <div class="pulse-entry">
    <div class="pulse-date">Jan 13, 2025</div>
    <div class="pulse-text">Attended a virtual workshop on MLOps best practices. Learned about new tools for model versioning and experiment tracking.</div>
  </div>
  
  <div class="pulse-entry">
    <div class="pulse-date">Jan 12, 2025</div>
    <div class="pulse-text">Fixed a bug in the production forecasting pipeline. The issue was related to timezone handling in the data preprocessing step.</div>
  </div>
  
  <div class="pulse-entry">
    <div class="pulse-date">Jan 11, 2025</div>
    <div class="pulse-text">Read a fascinating paper on diversity metrics in generative models. Thinking about how to apply similar concepts to our recommendation systems.</div>
  </div>
</div>

<style>
/* Regular Pulse styling - clean, minimal journal design */
.pulse-content {
  margin: 30px 0;
  max-width: 100%;
}

.pulse-entry {
  margin-bottom: 30px;
  padding-bottom: 25px;
  border-bottom: 1px solid #e5e7eb;
  position: relative;
}

.pulse-entry:last-child {
  border-bottom: none;
  margin-bottom: 0;
  padding-bottom: 0;
}

.pulse-date {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  font-size: 14px;
  font-weight: 600;
  color: #666;
  margin-bottom: 8px;
  letter-spacing: 0.02em;
  text-transform: uppercase;
}

.pulse-text {
  font-family: 'Georgia', 'Times New Roman', Times, serif;
  font-size: 16px;
  line-height: 1.7;
  color: #2c2c2c;
  margin: 0;
}

/* Timeline-style design option */
.pulse-entry::before {
  content: '';
  position: absolute;
  left: -20px;
  top: 5px;
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background-color: #8AA1B1;
  border: 2px solid #fff;
  box-shadow: 0 0 0 2px #e5e7eb;
}

@media (min-width: 768px) {
  .pulse-content {
    padding-left: 30px;
  }
  
  .pulse-entry::before {
    left: -10px;
  }
}

/* Mobile optimizations */
@media (max-width: 600px) {
  .pulse-content {
    padding-left: 0;
  }
  
  .pulse-entry::before {
    display: none;
  }
  
  .pulse-date {
    font-size: 13px;
    margin-bottom: 6px;
  }
  
  .pulse-text {
    font-size: 15px;
    line-height: 1.6;
  }
  
  .pulse-entry {
    margin-bottom: 20px;
    padding-bottom: 20px;
  }
}

/* Hover effect for entries */
.pulse-entry:hover .pulse-date {
  color: #8AA1B1;
  transition: color 0.2s ease;
}

/* Link styling within pulse entries */
.pulse-text a {
  color: #2563eb;
  text-decoration: none;
  border-bottom: 1px solid rgba(37, 99, 235, 0.3);
  transition: border-color 0.2s ease;
}

.pulse-text a:hover {
  border-bottom-color: #2563eb;
  color: #1d4ed8;
}

/* Code or technical terms */
.pulse-text code {
  background-color: #f1f5f9;
  padding: 2px 6px;
  border-radius: 3px;
  font-family: 'Monaco', 'Menlo', 'Courier New', monospace;
  font-size: 14px;
  color: #475569;
}
</style>

