---
layout: default
title: Home
---

<h1 class="hire-headline">Software Engineer / Generalist</h1>

<ul class="hire-points">
  <li>Worked across many stacks: frontend, backend, Android, ML, data pipelines, LLMs.</li>
  <li>Co-founded edumateengg.com, an edtech startup, at 19 and raised a seed round. It failed, and taught me a lot.</li>
  <li>Recommended on LinkedIn by the CEO of invideo AI, a company valued at $600M.</li>
  <li>Open source contributor.</li>
</ul>

<details class="accordion">
  <summary>Publications</summary>
  <p>
    You can also find these on my
    <a href="https://scholar.google.com/citations?user=jljZjeIAAAAJ&hl=en">Google Scholar profile</a>.
  </p>
  <ul>
    <li>
      <a href="https://link.springer.com/chapter/10.1007/978-981-96-4273-1_17">Automating Generation of Web GUI from a Design Image</a><br>
      A. Singh, A. Bansal. <em>International Conference on Intelligent Systems and Security</em>, pp. 223–234, 2024.
    </li>
    <li>
      <a href="https://www.proquest.com/openview/dbf12a2bbf207e3714c2fc8247b62f03/1?pq-origsite=gscholar&cbl=18750&diss=y">Automating Generation of Web GUI from a Design Image</a><br>
      A. J. Singh. <em>Arizona State University</em>, 2023.
    </li>
    <li>
      <a href="https://ieeexplore.ieee.org/document/9298263/">Rendering Videos from HTML Elements and Web Animations</a><br>
      A. Singh, A. Krishnan, S. Sharma. <em>2020 IEEE International Conference for Innovation in Technology (INOCON)</em>, 2020.
    </li>
  </ul>
</details>

<details class="accordion">
  <summary>Open Source Contributions</summary>
  <ul>
    <li>Contribution goes here.</li>
  </ul>
</details>

<details class="accordion">
  <summary>Blogs</summary>
  <ul>
    {%- for post in site.posts %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
      <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>
    </li>
    {%- endfor %}
  </ul>
</details>
