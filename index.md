---
layout: default
toc: true
toc_min_level: 2
toc_max_level: 4
---


<div class="intro-section">
  <img class="profile-picture intro-picture" src="asif_headshot.webp">
  <p>👋 Hi there! I'm <strong>Md Asif Bin Syed</strong>, a <em>Sr. Supply Chain Data Analyst</em> at <strong>The Home Depot</strong>, the world's leading home improvement retailer. Leading the development of offline <strong>reinforcement learning</strong> agents that reduce delivery failures by 4.5% ($6.5M retained revenue) and building ML Model for operational forecasting.</p>
</div>

<style>
.intro-section {
  margin-top: 20px;
  margin-left: 0;
  padding-left: 0;
  position: relative;
}

.intro-section p {
  margin: 0;
  padding: 0;
  text-align: justify;
}

.intro-picture {
  width: 160px;
  height: 160px;
  object-fit: cover;
  border-radius: 50%;
  float: right;
  margin-left: 15px;
  margin-bottom: 10px;
  margin-top: 0;
  shape-outside: circle(50%);
  display: block;
  clip-path: circle(50%);
}

/* Align with navbar title - account for navbar container padding */
@media (min-width: 501px) {
  .intro-section {
    margin-left: 0;
  }
}

@media (max-width: 600px) {
  .intro-picture {
    width: 120px;
    height: 120px;
    margin-right: 12px;
    margin-bottom: 8px;
  }
  
  .intro-section p {
    font-size: 15px;
  }
}
</style> 
I am a researcher specializing in **reinforcement learning**, **generative AI**, and **deep learning** across domains such as marine surveillance, medical diagnosis, supply chain optimization, and time series forecasting.
I have published in venues including **ICML Workshop'25**, **NeurIPS Workshop'25**, **IEEE conferences**, with contributions in time series foundation models, diversity quantification, and physics-informed neural networks.  

<div class="affiliations-section">
  <div class="affiliations-logos">
    <div class="affiliation-item">
      <img src="ICML-logo.svg" alt="ICML" class="affiliation-logo">
      <span class="affiliation-label">ICML</span>
    </div>
    <div class="affiliation-item">
      <img src="neurips-navbar-logo.svg" alt="NeurIPS" class="affiliation-logo">
      <span class="affiliation-label">NeurIPS</span>
    </div>
    <div class="affiliation-item">
      <img src="West-Virginia-University-WVU-Emblem.png" alt="West Virginia University" class="affiliation-logo">
      <span class="affiliation-label">WVU</span>
    </div>
    <div class="affiliation-item">
      <img src="georgia-tech-logo.png" alt="Georgia Tech" class="affiliation-logo">
      <span class="affiliation-label">Georgia Tech</span>
    </div>
    <div class="affiliation-item">
      <img src="Shahjalal_University_of_Science_and_Technology_logo.png" alt="Shahjalal University" class="affiliation-logo">
      <span class="affiliation-label">SUST</span>
    </div>
    <div class="affiliation-item">
      <img src="volvo.svg" alt="Volvo" class="affiliation-logo">
      <span class="affiliation-label">Volvo</span>
    </div>
    <div class="affiliation-item">
      <img src="the-home-depot.png" alt="The Home Depot" class="affiliation-logo">
      <span class="affiliation-label">Home Depot</span>
    </div>
  </div>
</div>

<style>
.affiliations-section {
  margin: 30px 0;
  padding: 20px 0;
}

.affiliations-logos {
  display: flex;
  flex-wrap: wrap;
  gap: 30px;
  align-items: center;
  justify-content: flex-start;
}

.affiliation-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  transition: transform 0.2s ease;
}

.affiliation-item:hover {
  transform: translateY(-2px);
}

.affiliation-logo {
  height: 50px;
  width: auto;
  max-width: 120px;
  object-fit: contain;
  filter: grayscale(20%);
  transition: filter 0.3s ease;
}

.affiliation-item:hover .affiliation-logo {
  filter: grayscale(0%);
}

.affiliation-label {
  font-size: 12px;
  color: #666;
  font-weight: 500;
  text-align: center;
}

@media (max-width: 600px) {
  .affiliations-logos {
    gap: 20px;
    justify-content: center;
  }
  
  .affiliation-logo {
    height: 40px;
    max-width: 100px;
  }
}
</style>

<blockquote style="margin-left: 3.5em;">
    <div style="display: flex; align-items: left; margin-left: -3.5em;">2025 - </div>
    <img src="the-home-depot.png" alt="Employer 1" style="width: 24px; height: 24px; margin-right: 10px;">
  <font size="3"> Working in <a href="https://www.homedepot.com" style="color: blue;">The Home Depot</a> leveraging machine learning and reinforcement learning to optimize our delivery network - using predictive models to forecast delivery times, route optimization algorithms to determine the most efficient delivery paths, and reinforcement learning to dynamically adjust delivery schedules based on real-time conditions.</font>
 <div style="display: flex; align-items: left; margin-left: -3.5em;">2024 - </div>
</blockquote>

<blockquote style="margin-left: 3.5em;">
    <img src="volvo.svg" alt="Employer 1" style="width: 24px; height: 24px; margin-right: 10px;">
  <font size="3">leverage machine learning in <a href="https://www.homedepot.com" style="color: blue;">Volvo</a>  to optimize the  load for carrier  and forecasting the demand for inboud deliveries</font>
 <div style="display: flex; align-items: left; margin-left: -3.5em;">2023 - </div>
</blockquote>

<main markdown="1">

<h2 id="publications">📄 Publications</h2>
<hr>
I have published and presented my work at prestigious conferences and journals, including Journal of Marine Science and Engineering, Sensors, IEEE conferences, and IISE, as well as workshops at various international venues.

<div class="publication-entry">
<img src="https://scholar.google.com/favicon.ico" alt="Google Scholar" style="width: 16px; height: 16px; vertical-align: middle; margin-right: 5px; display: inline-block;"> <strong>Zero-Shot Time-Series Forecasting: Do Time-Series FMs Outperform Domain-Agnostic FMs?</strong> · <em><strong>Syed, M. A. B.</strong>, et al. (2025)</em> · <strong>NeurIPS 2025 Workshop (BERT²S)</strong> · <a class="bracket-link" href="https://openreview.net/forum?id=HvbMOFV9zZ">OpenReview</a> · <span class="pill neurips">NeurIPS</span> <span class="pill deep">Deep Learning</span> <span class="pill timeseries">Time Series</span> <span class="pill foundation">Foundation Models</span><br>
<a href="#" class="abstract-toggle" onclick="toggleAbstract(event, 'abstract6')">Show Abstract</a>
<div class="abstract-content" id="abstract6" style="display: none;">
<strong>Abstract:</strong> Foundation models (FMs) have achieved major advances in language, vision, and speech. In parallel, time-series foundation models (TSFMs) have been developed to address forecasting tasks. A key question is whether TSFMs truly generalize to unseen time series data, and whether they perform better than general-purpose FMs from other domains in a zero-shot setting. We compare four TSFMs such as Chronos, Times-FM, TimeGPT, and MOMENTs with cross-domain FMs for text (GPT), audio (Whisper), and vision (ViT). For a systematic comparison, we use simple task-agnostic adapters to convert sequences into forecasts, without fine-tuning or changing the backbone models. All models are evaluated on nine diverse datasets that were unseen during training. Our results show that TSFMs perform best on most datasets, highlighting the benefit of temporal pretraining and time-aware design. Overall, the strong zero-shot performance of TSFMs suggests that they may represent a breakthrough comparable to BERT for time series forecasting. At the same time, large text-based models such as GPT remain surprisingly competitive, in some cases even surpassing TSFMs, highlighting the ability of general-purpose models to capture temporal patterns despite not being trained for this task. GitHub repository: https://github.com/anonymous4865/tsfms.
</div>
</div>

<div class="publication-entry">
<img src="https://scholar.google.com/favicon.ico" alt="Google Scholar" style="width: 16px; height: 16px; vertical-align: middle; margin-right: 5px; display: inline-block;"> <strong>Advancing Marine Surveillance: A Hybrid Approach of Physics Infused Neural Network for Enhanced Vessel Tracking Using Automatic Identification System Data</strong> · <em>Haque, T., <strong>Syed, M. A. B.</strong>, Das, S., &amp; Ahmed, I. (2024)</em> · <strong>Journal of Marine Science and Engineering, 12(11), 1913</strong> · <a class="bracket-link" href="https://doi.org/10.3390/jmse12111913">DOI</a> · <span class="pill marine">Marine AI</span> <span class="pill deep">Deep Learning</span> <span class="pill physics">Physics-Informed</span><br>
<a href="#" class="abstract-toggle" onclick="toggleAbstract(event, 'abstract1')">Show Abstract</a>
<div class="abstract-content" id="abstract1" style="display: none;">
<strong>Abstract:</strong> This paper presents a novel hybrid approach combining physics-informed neural networks with automatic identification system (AIS) data for enhanced vessel tracking in marine surveillance applications. Our methodology integrates domain knowledge from maritime physics with deep learning techniques to improve tracking accuracy and robustness. The proposed framework addresses key challenges in vessel trajectory prediction and association, demonstrating significant improvements over traditional methods through extensive experimental validation on real-world AIS datasets.
</div>
</div>

<div class="publication-entry">
<img src="https://scholar.google.com/favicon.ico" alt="Google Scholar" style="width: 16px; height: 16px; vertical-align: middle; margin-right: 5px; display: inline-block;"> <strong>Federated Learning in Manufacturing: A Systematic Review and Pathway to Industry 5.0</strong> · <em><strong>Syed, M. A. B.</strong>, Rhaman, Q., &amp; Sushil, S. (2023)</em> · <strong>IEEE STI 2023</strong> · <a class="bracket-link" href="https://ieeexplore.ieee.org/abstract/document/10464397">DOI</a> · <span class="pill federated">Federated Learning</span> <span class="pill manufacturing">Manufacturing</span> <span class="pill industry">Industry 5.0</span><br>
<a href="#" class="abstract-toggle" onclick="toggleAbstract(event, 'abstract2')">Show Abstract</a>
<div class="abstract-content" id="abstract2" style="display: none;">
<strong>Abstract:</strong> This systematic review explores the application of federated learning in manufacturing environments, examining its potential to enable Industry 5.0 transformation. We analyze current research trends, identify key challenges in distributed learning for industrial settings, and propose a comprehensive framework for implementing federated learning systems in manufacturing. Our analysis covers privacy-preserving machine learning, edge computing integration, and collaborative model training across multiple manufacturing facilities.
</div>
</div>

<div class="publication-entry">
<img src="https://scholar.google.com/favicon.ico" alt="Google Scholar" style="width: 16px; height: 16px; vertical-align: middle; margin-right: 5px; display: inline-block;"> <strong>A CNN-LSTM Architecture for Marine Vessel Track Association Using Automatic Identification System (AIS) Data</strong> · <em><strong>Syed, M. A. B.</strong>, &amp; Ahmed, I. (2023)</em> · <strong>Sensors, 23(14), 6400</strong> · <a class="bracket-link" href="https://doi.org/10.3390/s23146400">DOI</a> · <span class="pill marine">Marine AI</span> <span class="pill deep">Deep Learning</span> <span class="pill tracking">Track Association</span><br>
<a href="#" class="abstract-toggle" onclick="toggleAbstract(event, 'abstract3')">Show Abstract</a>
<div class="abstract-content" id="abstract3" style="display: none;">
<strong>Abstract:</strong> We propose a hybrid CNN-LSTM architecture for solving the track association problem in marine surveillance using AIS data. The model combines convolutional neural networks for spatial feature extraction with long short-term memory networks for temporal sequence modeling. Our approach effectively handles the challenges of vessel trajectory prediction and association in complex maritime environments, achieving superior performance compared to traditional methods on large-scale AIS datasets.
</div>
</div>

<div class="publication-entry">
<img src="https://scholar.google.com/favicon.ico" alt="Google Scholar" style="width: 16px; height: 16px; vertical-align: middle; margin-right: 5px; display: inline-block;"> <strong>Investigation of Polycystic Ovary Syndrome (PCOS) Diagnosis Using Machine Learning Approaches</strong> · <em>Habib, A. Z. S. B., <strong>Syed, M. A. B.</strong>, Islam, M. E., &amp; Tasnim, T. (2023)</em> · <strong>IEEE STI 2023</strong> · <a class="bracket-link" href="https://ieeexplore.ieee.org/abstract/document/10465079">DOI</a> · <span class="pill medical">Medical AI</span> <span class="pill deep">Deep Learning</span> <span class="pill diagnosis">Diagnosis</span><br>
<a href="#" class="abstract-toggle" onclick="toggleAbstract(event, 'abstract4')">Show Abstract</a>
<div class="abstract-content" id="abstract4" style="display: none;">
<strong>Abstract:</strong> This study investigates the application of machine learning techniques for diagnosing Polycystic Ovary Syndrome (PCOS), a common endocrine disorder affecting women. We evaluate multiple machine learning algorithms including support vector machines, random forests, and deep neural networks using clinical and biochemical parameters. Our results demonstrate the potential of ML-based diagnostic systems to assist healthcare professionals in early detection and accurate diagnosis of PCOS, potentially improving patient outcomes through timely intervention.
</div>
</div>

<div class="publication-entry">
<img src="https://scholar.google.com/favicon.ico" alt="Google Scholar" style="width: 16px; height: 16px; vertical-align: middle; margin-right: 5px; display: inline-block;"> <strong>Understanding AI Chatbot Adoption in Education: PLS-SEM Analysis of User Behavior Factors</strong> · <em>Hasan, M. R., Chowdhury, N. I., Rahman, M. H., <strong>Syed, M. A. B.</strong>, &amp; Ryu, J. (2023)</em> · <strong>Journal of Computers in Human Behavior: Artificial Humans</strong> · <a class="bracket-link" href="https://www.sciencedirect.com/science/article/pii/S2949882124000586">DOI</a> · <span class="pill nlp">NLP</span> <span class="pill education">Education</span> <span class="pill plssem">PLS-SEM</span><br>
<a href="#" class="abstract-toggle" onclick="toggleAbstract(event, 'abstract5')">Show Abstract</a>
<div class="abstract-content" id="abstract5" style="display: none;">
<strong>Abstract:</strong> This research employs Partial Least Squares Structural Equation Modeling (PLS-SEM) to investigate the factors influencing AI chatbot adoption in educational settings. We examine user behavior patterns, technology acceptance factors, and educational outcomes associated with chatbot implementation. Our findings reveal key determinants of adoption including perceived usefulness, ease of use, and educational value, providing insights for educators and technology developers seeking to enhance learning experiences through AI-powered conversational interfaces.
</div>
</div>

<a href="publications" class="btn btn-primary">View All Publications →</a>

<h2 id="technical-skills">Technical Skills</h2>
<hr>
<ul class="skills-list">
  <li><strong>Programming Languages:</strong> Python, R, SQL, C</li>
  <li><strong>ML/DL Frameworks:</strong> Scikit-learn, Keras, TensorFlow, PyTorch</li>
  <li><strong>Data Analysis:</strong> MS Excel, Tableau, Power BI, Minitab</li>
  <li><strong>Cloud Platforms:</strong> AWS, Azure ML, GCP, Vertex AI</li>
  <li><strong>MLOps & DevOps:</strong> Docker, Kubernetes, Jenkins, Git, GitHub Actions, MLflow, DVC, Weights & Biases</li>
  <li><strong>Databases:</strong> BigQuery, MySQL, PostgreSQL</li>
  <li><strong>Model Deployment:</strong> FastAPI, Flask, TensorFlow Serving, Model Monitoring, A/B Testing, CI/CD Pipelines</li>
  <li><strong>Other Tools:</strong> SAP Ariba, SAP MDCS, SolidWorks, MS Project, MS Access, CPLEX, HTML, CSS</li>
</ul>

<style>
.skills-list {
  list-style: none;
  padding: 0;
  margin: 10px 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Inter', 'Roboto', 'Helvetica Neue', Arial, sans-serif;
}

.skills-list li {
  margin-bottom: 8px;
  font-size: 14px;
  line-height: 1.6;
  color: #2c2c2c;
  padding-left: 0;
}

.skills-list li strong {
  color: #1a1a1a;
  font-weight: 600;
}

@media (max-width: 600px) {
  .skills-list li {
    font-size: 13px;
    line-height: 1.5;
    margin-bottom: 6px;
  }
}
</style>

<h2 id="whats-new" class="news-heading">📰 News and Updates</h2>
<hr>
<div class="news-content" style="max-height: 250px; overflow-y: auto; padding: 10px;">
  <ul>
    <li>
      <span>Nov 2024:</span>
      <span>Promoted to Sr. data analyst in supply chain at The Home Depot</span>
    </li>
    <li>
      <span>Oct 2024:</span>
      <span>Published a journal in Journal of Marine Science and Engineering: 
        <a href="https://www.mdpi.com/2077-1312/12/11/1913"><b>"Advancing Marine Surveillance: A Hybrid Approach of Physics Infused Neural Network for Enhanced Vessel Tracking Using Automatic Identification System Data."</b></a>
      </span>
    </li>
    <li>
      <span>Oct 2024:</span>
      <span>Awarded my first associate of the month by The Home Depot for discovering an opportunity to save around 2 M dollars by reducing extra pallets required</span>
    </li>
    <li>
      <span>Aug 2024:</span>
      <span>Published a journal in Journal of Computers in Human Behavior: Artificial Humans 
        <a href="https://www.sciencedirect.com/science/article/pii/S2949882124000586"><b>"Understanding AI Chatbot Adoption in Education: PLS-SEM Analysis of User Behavior Factors."</b></a>
      </span>
    </li>
    <li>
      <span>Jan 2024:</span>
      <span>Started my full-time position as a Data Analyst in supply chain at The Home Depot</span>
    </li>
    <li>
      <span>Dec 2023:</span>
      <span>Presented a paper on 
        <a href="https://ieeexplore.ieee.org/abstract/document/10465079"><b>"Investigation of Polycystic Ovary Syndrome (PCOS) Diagnosis Using Machine Learning Approaches"</b></a> 
        and 
        <a href="https://ieeexplore.ieee.org/abstract/document/10441152"><b>"A Deep Learning Approach for Satellite and Debris Detection: YOLO in Action"</b></a> 
        at the 2023 5th International Conference on Sustainable Technologies for Industry 5.0 (STI).
      </span>
    </li>
    <li>
      <span>Dec 2023:</span>
      <span>Presented a paper on 
        <a href="https://ieeexplore.ieee.org/abstract/document/10441258"><b>"Pediatric Bone Age Prediction Using Deep Learning"</b></a> 
        and 
        <a href="https://ieeexplore.ieee.org/abstract/document/10464397"><b>"Federated Learning in Manufacturing: A Systematic Review and Pathway to Industry 5.0"</b></a> 
        at the 2023 26th International Conference on Computer and Information Technology (ICCIT).
      </span>
    </li>
    <li>
      <span>Dec 2023:</span>
      <span>Completed my Master's in Industrial Engineering and submitted my thesis on  
        <a href="https://researchrepository.wvu.edu/cgi/viewcontent.cgi?article=12915&context=etd"><b>"Spatio-Temporal Deep Learning Approaches for Addressing Track Association Problem Using Automatic Identification System (AIS) Data"</b></a>
      </span>
    </li>
    <li>
      <span>July 2023:</span>
      <span>Awarded "Idea of the Month" at Volvo Trucks for implementing Power Automate and AI to extract invoice data, saving $200 K.</span>
    </li>
    <li>
      <span>July 2023:</span>
      <span>Published a journal in MDPI Sensors: 
        <a href="https://www.mdpi.com/1424-8220/23/14/6400"><b>"A CNN-LSTM Architecture for Marine Vessel Track Association Using AIS Data."</b></a>
      </span>
    </li>
    <li>
      <span>May 2023:</span>
      <span>Finalist in the QCRE Data Challenge for 
        <a href="https://arxiv.org/pdf/2309.13402.pdf"><b>"ML Algorithm Synthesizing Domain Knowledge for Fungal Spore Concentration Prediction."</b></a>
      </span>
    </li>
    <li>
      <span>April 2023:</span>
      <span>Submitted a paper to the IISE conference on 
        <a href="https://arxiv.org/abs/2304.01491"><b>"Multi-Model LSTM Architecture for Track Association Using AIS Data."</b></a>
      </span>
    </li>
    <li>
      <span>October 2022:</span>
      <span>Chaired a session at the INFORMS Annual Meeting on 
        <a href="https://meetings.informs.org/wordpress/indianapolis2022/"><b>"Advanced Machine Learning."</b></a>
      </span>
    </li>
  </ul>
</div>

<style>
/* Clean typography for homepage */
body {
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  line-height: 1.4;
  color: #333;
}

/* Social media badges - shields.io style matching the image */
.social-links {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin: 20px 0;
}

.social-badge {
  display: inline-flex;
  align-items: center;
  font-size: 11px;
  font-weight: 600;
  line-height: 1;
  color: #fff;
  text-decoration: none;
  border-radius: 0;
  padding: 4px 8px;
  background-color: #181717;
  position: relative;
  height: 20px;
  box-sizing: border-box;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial, sans-serif;
  transition: opacity 0.2s;
  white-space: nowrap;
}

.social-badge:hover {
  opacity: 0.85;
  text-decoration: none;
  color: #fff;
}

.social-badge .social-icon {
  display: inline-block;
  margin-right: 6px;
  width: 16px;
  height: 16px;
  flex-shrink: 0;
  vertical-align: middle;
}

.social-badge .social-icon svg {
  width: 100%;
  height: 100%;
  display: block;
}

.social-text {
  display: inline-block;
  font-size: 11px;
  line-height: 1;
}

/* Social badge colors matching the image */
.social-badge.scholar {
  background-color: #181717; /* Black */
}

.social-badge.scholar .social-icon {
  width: 16px;
  height: 16px;
}

.social-badge.linkedin {
  background-color: #0077B5; /* LinkedIn blue */
  padding-left: 8px;
}

.social-badge.linkedin .social-text {
  font-weight: 600;
}

.social-badge.arxiv {
  background-color: #181717; /* Black */
}

.social-badge.arxiv .social-icon {
  width: 14px;
  height: 14px;
  margin-right: 6px;
}

.social-badge.orcid {
  background-color: #181717; /* Black */
}

.social-badge.orcid .social-icon.orcid-icon {
  width: 16px;
  height: 16px;
  margin-right: 6px;
}

.social-badge.orcid .social-icon.orcid-icon circle {
  fill: #A6CE39;
}

.social-badge.orcid .social-icon.orcid-icon text {
  fill: #fff;
  font-size: 9px;
  font-weight: 700;
}

.research-highlights {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 12px;
  margin: 15px 0;
  padding: 15px;
  background-color: #f8fafc;
  border-radius: 6px;
  border: 1px solid #e2e8f0;
}

.highlight-stat {
  text-align: center;
}

.highlight-stat strong {
  display: block;
  font-size: 1.4em;
  color: #2563eb;
  margin-bottom: 3px;
  font-weight: 700;
}

.highlight-stat span {
  color: #64748b;
  font-size: 12px;
  font-weight: 500;
}

.research-areas {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin: 15px 0;
}

.area-tag {
  padding: 3px 8px;
  background-color: #f1f5f9;
  border-radius: 12px;
  font-size: 12px;
  color: #475569;
  border: 1px solid #e2e8f0;
  font-weight: 500;
}

.btn {
  display: inline-block;
  padding: 8px 16px;
  background-color: #2563eb;
  color: white;
  text-decoration: none;
  border-radius: 6px;
  font-weight: 500;
  font-size: 14px;
  transition: background-color 0.2s ease;
  margin-top: 10px;
}

.btn:hover {
  background-color: #1d4ed8;
  text-decoration: none;
  color: white;
}

/* Tighten up headings */
h2 {
  margin: 25px 0 10px 0;
  font-size: 18px;
  font-weight: 600;
  color: #1a1a1a;
}

/* Override for News heading to be more prominent */
h2.news-heading {
  margin: 25px 0 8px 0;
}

hr {
  margin: 8px 0 15px 0;
  border: none;
  border-top: 1px solid #e5e7eb;
}

/* Compact layout to match publications page */
main {
  max-width: 900px;
  margin: 0 auto;
  padding: 0 15px;
}

/* News and Updates section - academic serif typography */
.news-heading {
  font-family: 'Times New Roman', Times, Georgia, 'Palatino Linotype', 'Book Antiqua', Palatino, serif;
  font-size: 1.5em;
  font-weight: 700;
  color: #1a1a1a;
  margin-bottom: 8px;
  letter-spacing: -0.01em;
}

.news-content {
  font-family: 'Times New Roman', Times, Georgia, 'Palatino Linotype', 'Book Antiqua', Palatino, serif;
  font-size: 14px;
  line-height: 1.5;
  color: #2c2c2c;
}

.news-content ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

.news-content ul li {
  font-family: 'Times New Roman', Times, Georgia, 'Palatino Linotype', 'Book Antiqua', Palatino, serif;
  font-size: 14px;
  line-height: 1.5;
  margin-bottom: 0;
  color: #2c2c2c;
  padding-left: 0;
}

.news-content ul li span:first-child {
  font-weight: 700;
  margin-right: 8px;
  color: #1a1a1a;
  display: inline-block;
  vertical-align: top;
}

.news-content ul li span:not(:first-child) {
  font-weight: 400;
  color: #2c2c2c;
  display: inline-block;
  padding-left: 0;
  text-indent: 0;
}

/* Indent wrapped text to align with content, not date */
@media (min-width: 1px) {
  .news-content ul li {
    display: flex;
    align-items: flex-start;
  }
  
  .news-content ul li span:first-child {
    flex-shrink: 0;
    min-width: 90px;
  }
  
  .news-content ul li span:not(:first-child) {
    flex: 1;
    padding-left: 0;
  }
}

/* Mobile optimizations for News section */
@media (max-width: 600px) {
  .news-content {
    font-size: 12px;
    line-height: 1.4;
  }
  
  .news-content ul li {
    font-size: 12px;
    line-height: 1.4;
  }
  
  .news-content ul li span:first-child {
    min-width: 65px;
    margin-right: 6px;
    font-size: 12px;
  }
  
  .news-content ul li span:not(:first-child) {
    font-size: 12px;
  }
  
  .news-content ul li a {
    font-size: 12px;
  }
  
  .news-heading {
    font-size: 1.3em;
  }
}

.news-content ul li a {
  font-family: 'Times New Roman', Times, Georgia, 'Palatino Linotype', 'Book Antiqua', Palatino, serif;
  font-size: 14px;
  font-weight: 700;
  color: #1a1a1a;
  text-decoration: underline;
  text-decoration-color: rgba(0, 0, 0, 0.3);
}

.news-content ul li a:hover {
  text-decoration-color: #1a1a1a;
  color: #000;
}

/* Publications styling - matching publications page format */
/* Shield-style badge design like GitHub shields.io */
.pill {
  display: inline-flex;
  align-items: center;
  font-size: 11px;
  font-weight: 600;
  line-height: 1;
  margin: 0 4px 2px 0;
  color: #fff;
  text-decoration: none;
  border-radius: 0;
  padding: 4px 8px;
  background-color: #181717;
  position: relative;
  height: 20px;
  box-sizing: border-box;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Helvetica, Arial, sans-serif;
}

.pill::before {
  content: '';
  display: inline-block;
  width: 16px;
  height: 16px;
  margin-right: 6px;
  background-size: contain;
  background-repeat: no-repeat;
  background-position: center;
  vertical-align: middle;
}

.pill:hover { 
  opacity: 0.9;
  text-decoration: none;
  color: #fff;
}

/* Badge color schemes - using provided color palette */
/* Graphite #353535, White #FFFFFF, Pale Slate #D2D7DF, Silver #BDBBB0, Grey Olive #8A897C */
.pill.marine { 
  background-color: #353535; /* Graphite */
}
.pill.marine::before {
  content: '🌊';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.deep { 
  background-color: #8A897C; /* Grey Olive */
}
.pill.deep::before {
  content: '🧠';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.physics { 
  background-color: #353535; /* Graphite */
}
.pill.physics::before {
  content: '⚛️';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.education { 
  background-color: #8A897C; /* Grey Olive */
}
.pill.education::before {
  content: '📚';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.nlp { 
  background-color: #BDBBB0; /* Silver */
  color: #353535; /* Dark text for light background */
}
.pill.nlp::before {
  content: '💬';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.medical { 
  background-color: #353535; /* Graphite */
}
.pill.medical::before {
  content: '🏥';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.federated { 
  background-color: #8A897C; /* Grey Olive */
}
.pill.federated::before {
  content: '🔗';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.manufacturing { 
  background-color: #D2D7DF; /* Pale Slate */
  color: #353535; /* Dark text for light background */
}
.pill.manufacturing::before {
  content: '🏭';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.industry { 
  background-color: #BDBBB0; /* Silver */
  color: #353535; /* Dark text for light background */
}
.pill.industry::before {
  content: '🏭';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.tracking { 
  background-color: #8A897C; /* Grey Olive */
}
.pill.tracking::before {
  content: '📍';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.diagnosis { 
  background-color: #D2D7DF; /* Pale Slate */
  color: #353535; /* Dark text for light background */
}
.pill.diagnosis::before {
  content: '🔬';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.plssem { 
  background-color: #353535; /* Graphite */
}
.pill.plssem::before {
  content: '📊';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.neurips { 
  background-color: #353535; /* Graphite */
}
.pill.neurips::before {
  content: '🧠';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.timeseries { 
  background-color: #8A897C; /* Grey Olive */
}
.pill.timeseries::before {
  content: '📈';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

.pill.foundation { 
  background-color: #D2D7DF; /* Pale Slate */
  color: #353535; /* Dark text for light background */
}
.pill.foundation::before {
  content: '🔬';
  font-size: 12px;
  width: auto;
  height: auto;
  margin-right: 4px;
}

/* Bracket links */
a.bracket-link { 
  text-decoration: none; 
  color: #000;
  font-weight: 500;
}
a.bracket-link::before { content: "["; margin-right: 2px; }
a.bracket-link::after  { content: "]"; margin-left: 2px; }
a.bracket-link:hover {
  color: #333;
}

/* Publication entry styling */
.publication-entry {
  margin-bottom: 0;
  font-size: 14px;
  line-height: 1.4;
  color: #000;
}

.publication-entry strong {
  color: #000;
  font-weight: 600;
}

.publication-entry em {
  font-size: 13px;
  color: #000;
  font-style: italic;
}

.publication-entry em strong {
  text-decoration: underline;
}

/* Abstract toggle styling */
.abstract-toggle {
  color: #8AA1B1;
  text-decoration: none;
  font-size: 13px;
  font-weight: 500;
  cursor: pointer;
  margin-left: 5px;
}

.abstract-toggle:hover {
  color: #4A5043;
  text-decoration: underline;
}

.abstract-content {
  margin-top: 10px;
  padding: 10px;
  background-color: #f8f8f8;
  border-left: 3px solid #8AA1B1;
  font-size: 13px;
  line-height: 1.6;
  color: #333;
}

</style>

<script>
function toggleAbstract(event, abstractId) {
  event.preventDefault();
  const abstract = document.getElementById(abstractId);
  const toggle = event.target;
  
  if (abstract.style.display === 'none') {
    abstract.style.display = 'block';
    toggle.textContent = 'Hide Abstract';
  } else {
    abstract.style.display = 'none';
    toggle.textContent = 'Show Abstract';
  }
}
</script>

<blockquote style="border-left: 4px solid #000080; padding-left: 15px;">
  <p style="font-size: 18px; color: #000080; font-weight: bold;">
    Consistency is the key to self-improvement, and my GitHub activity serves as a powerful reminder of my commitment to continuous learning and coding progress.
  </p>
</blockquote>

<img src="https://ghchart.rshah.org/409ba5/Asifbinsyed" alt="GitHub Contributions Chart" />

</main>
