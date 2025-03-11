---
layout: post
title: How GitHub Actions work 
date: 2025-03-01 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Shillong, Small, City]
---


🔹 GitHub Actions Workflow: Design Patterns & Concepts
GitHub Actions workflows use YAML for configuration but execute commands using shell scripts, Python, JavaScript, or other scripting languages. Here’s a breakdown of design patterns and concepts involved:

📌 1. Design Pattern: Observer Pattern
Concept: GitHub Actions listens for events (like push, pull_request, or schedule) and triggers workflows based on them.
How it applies:
The GitHub Actions runner acts as an observer waiting for events.
When an event occurs (e.g., push to main), the workflow triggers automatically.
Example:
{% highlight ruby %}
on:
  push:
    branches:
      - main
{% endhighlight %}

This observes for push events on the main branch and reacts by executing a workflow.

📌 2. Concept: Declarative Configuration (Infrastructure as Code)
YAML defines the workflow declaratively, meaning it describes what to do rather than how.
Example:
{% highlight ruby %}
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Set up Node.js
        uses: actions/setup-node@v3
{% endhighlight %}

You don't write imperative scripts for environment setup; you declare what should be installed.

📌 3. State Machine / Workflow Orchestration
GitHub Actions executes steps sequentially or in parallel based on dependencies.
This follows a state machine model, where:
Each step has an input state (previous step’s output).
The next step runs only if conditions are met.
Example (Conditional Execution):
{% highlight ruby %}
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests
        run: npm test
      - name: Deploy
        if: success()
        run: echo "Deploying app..."
if: success() ensures deployment only runs if tests pass.
{% endhighlight %}


📌 4. Chain of Responsibility (Job Dependencies)
Jobs can depend on each other, forming a pipeline.
Example:
{% highlight ruby %}
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building project"
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
{% endhighlight %}

deploy only runs after build completes (needs: build).
This is similar to middleware pipelines or chain of responsibility patterns.

📌 5. Strategy Pattern (Matrix Builds)
GitHub Actions supports dynamic job execution based on a strategy.
Example (Run tests on multiple versions of Node.js):

{% highlight ruby %}
jobs:
  test:
    strategy:
      matrix:
        node-version: [14, 16, 18]
    runs-on: ubuntu-latest
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
{% endhighlight %}
The workflow dynamically runs the job three times, each with a different Node.js version.

📌 6. Microservices & Event-Driven Architecture
Each job runs independently in its own container or VM.
Event-driven: Workflows trigger based on GitHub events (push, pull_request, schedule, etc.).
Example: Scheduled workflows using cron jobs:

{% highlight ruby %}
on:
  schedule:
    - cron: '0 2 * * *'  # Runs every day at 2 AM UTC
{% endhighlight %}
This makes GitHub Actions useful for CI/CD pipelines, backups, and automation.

>🚀 Summary
    Concept / Pattern	How It Applies to GitHub Actions
    Observer Pattern	Listens for push, pull_request, schedule events
    Declarative Configuration	YAML defines workflows, avoiding imperative scripts
    State Machine / Workflow	Steps/jobs depend on previous states (if: success())
    Chain of Responsibility	Jobs depend on others (needs: build)
    Strategy Pattern	Runs tests on multiple configurations (matrix builds)
    Microservices Architecture	Jobs run independently in containers/VMs
    Event-Driven Architecture	Triggers based on GitHub events

>🌍 Languages Used
    YAML → Defines workflow configuration.
    Bash / Shell scripting → Common for build & deploy steps.
    JavaScript (Node.js) & Python → Used in custom GitHub Actions.
    Docker → Some workflows run inside Docker containers.
