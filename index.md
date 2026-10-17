---
layout: default
title: Home
---

<section class="intro">
  <h1 class="hire-headline">Software Engineer / Generalist</h1>

  <ul class="hire-points">
    <li>Worked across many stacks: frontend, backend, Android, ML, data pipelines, LLMs.</li>
    <li>Co-founded edumateengg.com, an edtech startup, at 19 and raised a seed round. It failed, and taught me a lot.</li>
    <li>Recommended on LinkedIn by the CEO of invideo AI, a company valued at $600M.</li>
    <li>Open source contributor.</li>
  </ul>
</section>

<details class="accordion">
  <summary><span>Experience</span><span class="count">3</span></summary>
  <ul class="item-list">
    <li>
      <span class="item-title">Senior Software Engineer · PayPal</span>
      <span class="item-meta">2023 – Present</span>
      <span class="item-desc">Delivered 1 project per year.</span>
    </li>
    <li>
      <span class="item-title">Software Engineer · invideo AI</span>
      <span class="item-meta">2018 – 2021</span>
      <span class="item-desc">Delivered 4 features per year.</span>
    </li>
    <li>
      <span class="item-title">Co-Founder · edumateengg.com</span>
      <span class="item-meta">2016 – 2018</span>
      <span class="item-desc">Delivered an e-learning Android app from scratch.</span>
    </li>
  </ul>
</details>

<details class="accordion">
  <summary><span>Publications</span><span class="count">2</span></summary>
  <ul class="item-list">
    <li>
      <a href="https://link.springer.com/chapter/10.1007/978-981-96-4273-1_17">Automating Generation of Web GUI from a Design Image</a>
      <span class="item-meta">A. Singh, A. Bansal · <em>International Conference on Intelligent Systems and Security</em>, pp. 223–234 · 2024</span>
    </li>
    <li>
      <a href="https://ieeexplore.ieee.org/document/9298263/">Rendering Videos from HTML Elements and Web Animations</a>
      <span class="item-meta">A. Singh, A. Krishnan, S. Sharma · <em>IEEE International Conference for Innovation in Technology (INOCON)</em> · 2020</span>
    </li>
  </ul>
  <p class="accordion-note">
    Also on <a href="https://scholar.google.com/citations?user=jljZjeIAAAAJ&hl=en">Google Scholar</a>.
  </p>
</details>

<details class="accordion">
  <summary><span>Open Source Contributions</span><span class="count" id="oss-count">…</span></summary>
  <ul class="item-list" id="oss-list">
    <li class="item-meta">Loading from GitHub…</li>
  </ul>
</details>

<script>
  // Fill the open source section from merged PRs to the repos listed below.
  (function () {
    var user = "ajiteshsingh";
    var shown = ["apache/iceberg-go", "pandas-dev/pandas"];
    var prsUrl = "https://github.com/pulls?q=is%3Apr+is%3Amerged+author%3A" + user;
    var list = document.getElementById("oss-list");
    var count = document.getElementById("oss-count");
    var query = "author:" + user + " type:pr is:merged -user:" + user;
    fetch("https://api.github.com/search/issues?per_page=100&q=" + encodeURIComponent(query))
      .then(function (res) {
        if (!res.ok) throw new Error(res.status);
        return res.json();
      })
      .then(function (data) {
        var repos = {};
        data.items.forEach(function (pr) {
          var name = pr.repository_url.split("/repos/")[1];
          if (shown.indexOf(name) === -1) return;
          repos[name] = (repos[name] || 0) + 1;
        });
        var names = Object.keys(repos).sort(function (a, b) {
          return repos[b] - repos[a];
        });
        count.textContent = names.length;
        list.innerHTML = "";
        names.forEach(function (name) {
          var n = repos[name];
          var li = document.createElement("li");
          var a = document.createElement("a");
          a.href = "https://github.com/" + name + "/pulls?q=is%3Apr+is%3Amerged+author%3A" + user;
          a.textContent = name;
          var meta = document.createElement("span");
          meta.className = "item-meta";
          meta.textContent = n + (n === 1 ? " merged PR" : " merged PRs");
          li.appendChild(a);
          li.appendChild(meta);
          list.appendChild(li);
        });
      })
      .catch(function () {
        count.textContent = "–";
        list.innerHTML = '<li class="item-meta">Couldn\'t load from GitHub right now. ' +
          '<a href="' + prsUrl + '">See my merged pull requests</a>.</li>';
      });
  })();
</script>

<details class="accordion">
  <summary><span>Blogs</span><span class="count">{{ site.posts | size }}</span></summary>
  <ul class="item-list">
    {%- for post in site.posts %}
    <li>
      {%- if post.external_url %}
      <a href="{{ post.external_url }}" target="_blank" rel="noopener">{{ post.title | escape }} ↗</a>
      <span class="item-meta">{{ post.date | date: "%b %-d, %Y" }} · {{ post.external_site }}</span>
      {%- else %}
      <a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
      <span class="item-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
      {%- endif %}
    </li>
    {%- endfor %}
  </ul>
</details>
