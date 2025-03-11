---
layout: post
title: how to add google analytics to a jekyll web page for SEO 
date: 2025-01-11 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Shillong, Small, City]
---

Adding Google Analytics to your Jekyll site for SEO tracking is straightforward. Here’s a step-by-step guide to properly integrate Google Analytics in a way that works with GitHub Pages' Content Security Policy (CSP).

✅ Step 1: Get Your Google Analytics Tracking ID
Go to Google Analytics and sign in.
Click on Admin (⚙️) → Create a Property.
Select Web as the platform.
Enter your website URL (e.g., https://yourwebsite.github.io).
Click Create, and copy the Measurement ID (looks like G-XXXXXXXXXX).
✅ Step 2: Add Google Analytics to Your Jekyll Site
Since GitHub Pages blocks inline scripts, use the recommended external script.

Method 1: Using _includes/ (Best Practice)
Create a new file _includes/google-analytics.html
Paste this Google Analytics script inside _includes/google-analytics.html:


{% highlight ruby %}
<!-- Google Analytics -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());

  gtag('config', 'G-XXXXXXXXXX'); // Replace with your actual Google Analytics ID
</script>
{% endhighlight %}


Include this file in _layouts/default.html before the closing </body> tag:
html
Copy
Edit
{% include google-analytics.html %}
📌 Why this works?

Avoids inline script blocking (since gtag.js is external).
Ensures analytics loads on every page.
SEO-friendly (tracks user activity).
✅ Step 3: Commit and Push Your Changes
Run these commands:

sh
Copy
Edit
git add _includes/google-analytics.html
git add _layouts/default.html
git commit -m "Added Google Analytics for SEO tracking"
git push origin main
✅ Step 4: Verify That Google Analytics Works
Open Google Analytics → Admin
Click Realtime → Overview
Visit your site in another browser or incognito mode
You should see live tracking!
🚀 Bonus: Improve SEO with Jekyll Plugins
Google Analytics is great for tracking visitors, but for better SEO:

Add meta tags dynamically with jekyll-seo-tag:
yaml
Copy
Edit
plugins:
  - jekyll-seo-tag
Then, in your _layouts/default.html:
html
Copy
Edit
{% seo %}
Submit your sitemap to Google Search Console:
Generate a sitemap with:
yaml
Copy
Edit
plugins:
  - jekyll-sitemap
Submit sitemap.xml to Google Search Console.
🎯 Final Summary
✔ Use Google’s recommended script (gtag.js) instead of inline scripts.
✔ Add it in _includes/google-analytics.html and include it in _layouts/default.html.
✔ Push your changes and verify tracking in Google Analytics.
✔ For better SEO, use jekyll-seo-tag and jekyll-sitemap.

Let me know if you need help! 🚀😊







