---
title: Results Report
permalink: /model/report/
sidebar:
  nav: "model"
---

<div class="page-report">
  <p><a href="/model/assessment/">Retake assessment</a></p>
  <div id="report-summary"></div>
  <h2>Your assessment report</h2>
  <p>This report summarizes your responses from the self-assessment and highlights where your practice is already strong and where it needs attention.</p>

  <div id="report-data" style="display:none">
    {% assign areas = site.areas | sort:'weight' | reverse %}
    {% for area in areas %}
      {% assign area_cards = site.cards | where:"area", area.slug %}
      {% for card in area_cards %}
        {% assign praise = '' %}
        {% assign remedy = '' %}
        {% if card.assessment and card.assessment.size > 0 %}
          {% assign first_assessment = card.assessment | first %}
          {% assign praise = first_assessment.praise %}
          {% assign remedy = first_assessment.remedy %}
        {% endif %}
        <div
          class="report-card"
          data-slug="{{ card.slug }}"
          data-title="{{ card.title | escape }}"
          data-url="{{ card.url }}"
          data-area="{{ area.slug }}"
          data-area-title="{{ area.title | escape }}"
          data-praise="{{ praise | escape }}"
          data-remedy="{{ remedy | escape }}"
        ></div>
      {% endfor %}
    {% endfor %}
  </div>

  <div id="report-output"></div>
</div>

<script>
  const cards = Array.from(document.querySelectorAll('.report-card')).map((element) => ({
    slug: element.dataset.slug,
    title: element.dataset.title,
    url: element.dataset.url,
    area: element.dataset.area,
    areaTitle: element.dataset.areaTitle,
    praise: element.dataset.praise,
    remedy: element.dataset.remedy
  }));

  function normalizeCardSlug(key) {
    return key.replace(/-\d+$/, '');
  }

  function readSavedScores() {
    if (typeof window === 'undefined' || !window.localStorage) {
      return {};
    }

    const entries = Object.keys(window.localStorage)
      .filter((key) => key !== 'assessment_complete')
      .map((key) => ({
        key,
        value: Number(window.localStorage.getItem(key))
      }))
      .filter((entry) => Number.isFinite(entry.value));

    console.log('assessment storage values', entries);

    const averages = {};
    entries.forEach((entry) => {
      const cardSlug = normalizeCardSlug(entry.key);
      if (!averages[cardSlug]) {
        averages[cardSlug] = [];
      }
      averages[cardSlug].push(entry.value);
    });

    return Object.fromEntries(Object.entries(averages).map(([cardSlug, values]) => {
      const average = values.reduce((sum, value) => sum + value, 0) / values.length;
      return [cardSlug, average];
    }));
  }

  function readTotalScore() {
    if (typeof window === 'undefined' || !window.localStorage) {
      return null;
    }

    const score = window.localStorage.getItem('assessment_score') || 0;
    const value = Number(score);
    return Number.isFinite(value) ? value : null;
  }

  function renderReport() {
    const savedScores = readSavedScores();
    const totalScore = readTotalScore();
    const groups = {
      strength: [],
      opportunity: [],
      neutral: []
    };

    cards.forEach((card) => {
      const rawScore = savedScores[card.slug];
      const score = typeof rawScore === 'number' ? rawScore : Number(rawScore);
      let bucket = 'neutral';

      if (Number.isFinite(score)) {
        if (score >= 4) {
          bucket = 'strength';
        } else if (score <= 2) {
          bucket = 'opportunity';
        }
      }

      groups[bucket].push({
        ...card,
        score
      });
    });

    const output = document.getElementById('report-output');
    const summary = document.getElementById('report-summary');
    if (summary) {
      summary.innerHTML = totalScore === null
        ? '<p>No total score was saved yet.</p>'
        : `<p><strong>Total score:</strong> ${totalScore}%</p>`;
    }

    const sections = [
      {
        key: 'strength',
        heading: 'Strengths',
        renderBody: (item) => `<p>${item.praise || 'Keep building on this practice.'}</p>`
      },
      {
        key: 'opportunity',
        heading: 'Opportunities',
        renderBody: (item) => `<p>${item.remedy || 'Review this practice and take one incremental action.'}</p>`
      },
      {
        key: 'neutral',
        heading: 'Areas to watch',
        renderBody: () => ''
      }
    ];

    const html = sections.map((section) => {
      const items = groups[section.key] || [];
      if (!items.length) {
        return '';
      }

      const listItems = items.map((item) => {
        const body = section.renderBody(item);
        return `<li><strong><a href="${item.url}">${item.title}</a></strong>${body}</li>`;
      }).join('');

      return `<h3>${section.heading}</h3><ul>${listItems}</ul>`;
    }).join('');

    output.innerHTML = html || '<p>No assessment values were found yet.</p>';
  }

  renderReport();
</script>
