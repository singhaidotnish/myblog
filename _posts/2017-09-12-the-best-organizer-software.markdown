---
layout: post
title: "Simple English Test Maker"
date: 2024-11-30
---

# Simple English Test Maker

Welcome to the **Simple English Test Maker**! This section helps you create easy and fun English tests to improve vocabulary and grammar skills. Just have fun

---

### How It Works:

1. **Pick a Test Type**:
   - **Multiple Choice Questions**
   - **Fill-in-the-Blank**
   - **True or False**

2. **Add Your Words and Definitions**:
   Enter your custom words and their meanings, and we’ll create questions for you automatically.

---

### Example Test:

#### Word List:
- **Dog**: A domestic animal known as man’s best friend.
- **Run**: To move quickly on foot.
- **Bright**: Giving off lots of light.

#### Generated Questions:

**Multiple Choice Question**:  
What does "Bright" mean?  
- A) To be clever.  
- B) Giving off lots of light.  
- C) Running fast.

**Fill-in-the-Blank**:  
A ________ is a loyal pet to humans.  
(Answer: Dog)

**True or False**:  
"Run" means to sit quietly.  
(Answer: False)

---

### Create Your Own Test:

Use the form below to input words and definitions, and we’ll generate questions for you:

```html
<form id="test-maker">
  <label for="word">Enter Word:</label>
  <input type="text" id="word" name="word" required>

  <label for="definition">Enter Definition:</label>
  <input type="text" id="definition" name="definition" required>

  <button type="submit">Generate Test</button>
</form>

<div id="generated-test"></div>

<script>
  document.getElementById("test-maker").addEventListener("submit", function(event) {
    event.preventDefault();
    const word = document.getElementById("word").value;
    const definition = document.getElementById("definition").value;

    document.getElementById("generated-test").innerHTML = `
      <h3>Generated Test:</h3>
      <p><b>Word:</b> ${word}</p>
      <p><b>Definition:</b> ${definition}</p>
      <p><b>Question:</b> What does "${word}" mean?</p>
      <ul>
        <li>${definition}</li>
        <li>Option 2</li>
        <li>Option 3</li>
      </ul>
    `;
  });
</script>
