---
layout: default
title: "CA Sakshi Jain Journal and Blog"
description: "Expert insights on US GAAP, IFRS, and Ind AS from a practicing Chartered Accountant. Technical analysis, CFO strategies, and audit expertise for global finance professionals."
---

{: .hero-section}
## 🧭 The Journal of a CA: Navigate Finance & Strategy with Me

**by [Sakshi Jain, CA](https://www.linkedin.com/in/casakshijain907/)**

{{ site.tagline }}

{% assign years = 'now' | date: "%Y" | minus: site.author.career_start_year %}
Welcome to my corner of the internet! I'm sharing what I'm learning about global accounting standards, audit excellence, and strategic finance as I grow through {{ years }}+ years as a practicing Chartered Accountant.

---

### 📖 What is this site about?

This is my digital garden and personal blog—a space where I document my journey and insights as a **Chartered Accountant** navigating through **Audit and Global Financial Reporting**.

Here, I share technical deep-dives, real experiences, and strategic thoughts. My goal? To help fellow finance professionals bridge the gap between **compliance-based accounting** and **forward-looking business strategy**. If you're looking to evolve your analytical and leadership skills, you're in the right place.

### 🎯 Topics I'm Writing About:

* **The Reporting Bedrock:** I dive deep into **US GAAP, IFRS, and Ind AS**—helping you navigate the complexities of global accounting standards and technical financial reporting.
* **Audit & Governance:** Drawing from my {{ years }}+ years in auditing, I share insights on data integrity, internal controls, and building robust financial hygiene.
* **The Leadership Pivot:** Practical guides on my journey from "Historical Recording" to "Predictive Forecasting"—covering driver-based models, scenario planning, and strategic thinking.
* **CFO Roadmap:** I'm exploring what it takes to lead a modern finance function: the soft skills, commercial acumen, and tech-stack mastery that separate good from great.

### 👤 About the Author

While I use this blog to share insights and learnings, you'll find my complete professional background, certifications, and project portfolio on my dedicated profile page:

👉 **[View My Professional Profile & Portfolio](https://ca-sakshi-jain.github.io/ca-sakshi-jain/)**

## Recent Posts

{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url }}) <small>({{ post.date | date: "%b %d, %Y" }})</small>
{% endfor %}
