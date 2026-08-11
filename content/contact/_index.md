---
title: "Contact"
description: "Get in touch with Steve Macenski"
---

**Email:** stevenmacenski [at] gmail [dot] com

**Location:** San Francisco, CA

I try to give my time generously but please keep it brief.

<form id="contact-form" class="contact-form">
  <div class="form-group">
    <label for="name">Name</label>
    <input type="text" id="name" name="name" required>
  </div>
  <div class="form-group">
    <label for="email">Email</label>
    <input type="email" id="email" name="email" required>
  </div>
  <div class="form-group">
    <label for="message">Message</label>
    <textarea id="message" name="message" required></textarea>
  </div>
  <div class="form-honeypot" aria-hidden="true">
    <label for="website">Website</label>
    <input type="text" id="website" name="website" tabindex="-1" autocomplete="off">
  </div>
  <button type="submit" class="form-submit">Send</button>
  <p id="form-status" class="form-status"></p>
</form>

<script>
document.getElementById('contact-form').addEventListener('submit', function(e) {
  e.preventDefault();
  var status = document.getElementById('form-status');
  var btn = this.querySelector('.form-submit');
  btn.disabled = true;
  btn.textContent = 'Sending...';

  // Replace this URL with your Google Apps Script Web App URL
  var SCRIPT_URL = 'https://script.google.com/macros/s/AKfycbzyxgm2uc3DJVYoiqjgYrhrXLiZUIrTiNMPE4XhszFfpkltxHxWhA0qmMvg4-kFPnAr-Q/exec';

  var data = {
    name: document.getElementById('name').value,
    email: document.getElementById('email').value,
    message: document.getElementById('message').value,
    website: document.getElementById('website').value
  };

  if (SCRIPT_URL === 'YOUR_GOOGLE_APPS_SCRIPT_URL') return;

  fetch(SCRIPT_URL, {
    method: 'POST',
    mode: 'no-cors',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  }).then(function() {
    status.textContent = 'Thank you! Your message has been sent.';
    status.style.color = '#16a34a';
    document.getElementById('contact-form').reset();
    setTimeout(function() { btn.disabled = false; btn.textContent = 'Send'; }, 5000);
  }).catch(function() {
    status.textContent = 'Something went wrong. Please email me directly.';
    status.style.color = '#dc2626';
    setTimeout(function() { btn.disabled = false; btn.textContent = 'Send'; }, 5000);
  });
});
</script>
