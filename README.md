# Vines of Life — Come & Abide

A free, self-paced program of emotional healing, listening partnership, and community — rooted in love, open to all.

## What this is

A single-file web program (`index.html`) that walks participants through ten modules of inner work, emotional healing, and relational practice. It is Christian-rooted, welcoming to people of any or no faith, and designed for both individual and small-group use.

Every module includes:

- **Teaching** — the core idea, explained plainly
- **A Scripture anchor** — one passage to sit with
- **Reflection journal** — private, saved in the browser
- **A 7-day "Live It" practice** — daily actions that put the teaching in your body
- **A Listening Partnership exercise** — structured time with another person, taking turns being heard
- **Group discussion questions** — for pairs or small groups
- **A Commitment card** — one sentence to carry forward

## The modules

1. Love Abundantly — The Root
2. Know Thy Self
3. The Three Domains of Learning
4. The Pathology of Dis-functionality
5. Beginning a Listening Partnership
6. Discharge — Letting Feelings Move
7. Self-Regulation & Inspiration
8. Connectivity — Empathy & Forgiveness
9. Social Skill & Community
10. My Goals for 20 Years

Plus a Welcome orientation and a closing Commissioning.

## How it works technically

- **One file, no dependencies.** All HTML, CSS, and JavaScript are inline in `index.html`.
- **Privacy by design.** All journal entries, practice notes, and progress are stored in the participant's browser via `localStorage`. Nothing is sent to any server.
- **Export / Import.** Participants can download their journal as JSON and restore it later.
- **Print-friendly.** Each module has clean print CSS.

## Running locally

Open `index.html` in any modern browser (Chrome, Firefox, Safari, Edge). No build step, no server required.

## Hosting

Because it's a single static file, you can host it anywhere:

- **GitHub Pages** — enable Pages on this repo's `main` branch, root folder. The site will be at `https://shaneclary.github.io/vinesoflife/`.
- **Netlify / Cloudflare Pages / Vercel** — point at this repo, no build command, publish directory is the repo root.
- **Any web host** — upload `index.html`.

## Newsletter setup

The site has a "Stay in touch" signup form that posts to a **Google Form** you own. Responses land in a Google Sheet you control. No server, no database, no cost. Until you configure it, the form is hidden.

### One-time setup (about 5 minutes)

1. **Create the Google Form.**
   Go to [forms.google.com](https://forms.google.com) → blank form → title it "Vines of Life — Keep me updated." Add two short-answer fields:
   - `Name` (required)
   - `Email` (required — under field options, toggle *Response validation → Text → Email*)

2. **Connect a Google Sheet for responses.**
   Click the **Responses** tab in the form editor → the green Sheets icon → "Create a new spreadsheet." Every signup now appears as a row in that Sheet.

3. **Get the form's submit URL and field IDs.**
   In the form editor, click the three-dot menu (top-right) → **Get pre-filled link**. Fill the fields with *anything* (e.g., "Name" and `a@a.com`) and click **Get link**. Copy the URL. It will look like:

   ```
   https://docs.google.com/forms/d/e/1FAIpQLSxxxxxx/viewform?usp=pp_url&entry.1234567890=Name&entry.9876543210=a%40a.com
   ```

4. **Extract the three values** you need for configuration:
   - **Form action URL** — take the URL up to `/viewform` and replace `viewform` with `formResponse`:
     ```
     https://docs.google.com/forms/d/e/1FAIpQLSxxxxxx/formResponse
     ```
   - **Name field entry ID** — the `entry.NNNNNNNNNN` that appeared *before* the fake name.
   - **Email field entry ID** — the `entry.NNNNNNNNNN` that appeared *before* the fake email.

5. **Paste them into `index.html`.**
   Near the top of the `<script>` block, find:
   ```js
   const NEWSLETTER = {
     enabled: false,
     formAction: "",
     fields: {
       name:  "entry.XXXXXXXXXX",
       email: "entry.XXXXXXXXXX",
     },
   };
   ```
   Change to:
   ```js
   const NEWSLETTER = {
     enabled: true,
     formAction: "https://docs.google.com/forms/d/e/1FAIpQLSxxxxxx/formResponse",
     fields: {
       name:  "entry.1234567890",
       email: "entry.9876543210",
     },
   };
   ```

6. **Commit and push.**
   ```
   git add index.html
   git commit -m "Enable newsletter signup via Google Forms"
   git push
   ```
   The form appears on the Welcome screen within ~30 seconds.

### Also get every signup as an email (recommended)

The Sheet is a good record, but it's easier to act on signups if each one arrives in your inbox. Add a **Google Apps Script trigger** on the form — free, stays inside Google, no third-party service needed.

1. Open the form for editing at [forms.google.com](https://forms.google.com).
2. Three-dot menu (top-right) → **Script editor**. A code editor opens in a new tab.
3. Replace the placeholder code with:

   ```javascript
   function onVolFormSubmit(e) {
     const v = e.namedValues || {};
     const name  = (v["Name"]  || [""])[0];
     const email = (v["Email"] || [""])[0];
     const body =
       "New Vines of Life signup\n" +
       "=========================\n\n" +
       "Name:     " + name  + "\n" +
       "Email:    " + email + "\n\n" +
       "Received: " + new Date().toLocaleString();
     MailApp.sendEmail({
       to:      "vinesoflife@aol.com",
       subject: "Vines of Life — new signup: " + name,
       body:    body,
       replyTo: email
     });
   }
   ```

4. Save the project (name it "Vines of Life form emails").
5. Click the **clock icon** in the left rail (Triggers) → **+ Add Trigger**:
   - Function: `onVolFormSubmit`
   - Event source: **From form**
   - Event type: **On form submit**
   - Save.
6. Approve the permissions prompt (it will warn about an "unverified app" — that's normal for your own scripts; click *Advanced → Go to project*).
7. Submit a test signup. You should get an email at `vinesoflife@aol.com` within ~10 seconds.

Quota: free Google accounts allow 100 Apps Script emails/day — far more than any newsletter signup form needs.

### Testing

Visit the live site, sign up with a test email, then check your Google Sheet — you should see a new row. If nothing appears:
- Confirm `enabled: true`.
- Confirm `formAction` ends in `/formResponse` (not `/viewform`).
- Confirm `entry.XXXX` IDs match the ones from your pre-filled link.
- Open the browser devtools Network tab during submit — you should see a POST to `docs.google.com/forms/...` that returns status `0` (this is normal; Google Forms doesn't send CORS headers).

### Alternative providers (if you outgrow Google Forms)

The form posts a standard `application/x-www-form-urlencoded` body. To switch providers, change `formAction` and the field names:

- **Mailchimp:** use the embedded form's `action` URL; fields are typically `FNAME` and `EMAIL` (different from `entry.*`).
- **Buttondown:** POST to `https://buttondown.email/api/emails/embed-subscribe/<your-username>`; field is `email`.
- **ConvertKit (Kit):** POST to `https://app.kit.com/forms/<id>/subscriptions`; fields are `name` and `email_address`.
- **Formspree:** POST to `https://formspree.io/f/<form-id>`; fields are free-form.

Each of these uses a similar `no-cors` submit pattern to Google Forms; you shouldn't need code changes beyond the `NEWSLETTER` config.

## Privacy & data

- **Participant journals** are stored in each reader's own browser (`localStorage`). They never leave the user's device. The site never uploads journal data anywhere.
- **Newsletter signups** (if enabled) go directly from the reader's browser to Google (or whichever provider you configured). The website itself never touches this data either.
- **GitHub Pages traffic analytics** are available under the repo's **Insights → Traffic** tab (aggregate page views and referrers, no personal data).

## Sources & attribution

The teaching draws on three streams, all credited on the in-site **Sources & Attribution** page:

1. **Vines of Life / IDmP (Interactive Discipline management Platform)** — the original framework, diagrams, listening-partnership structure, feelings-awareness approach, and vision exercise.
2. **Re-evaluation Counseling / Co-Counseling** — ideas on emotional *discharge* and *listening partnership* originated with Harvey Jackins and are documented in *Fundamentals of Co-Counseling Manual* and other publications from **Rational Island Publishers**. Those works are referenced and briefly quoted for educational use, with attribution. Vines of Life is not affiliated with the Re-evaluation Counseling Communities.
3. **Scripture** — public-domain English translations (KJV / WEB).

See the **Sources & Attribution** page in the site itself for the full credits and recommended reading.

## License

The Vines of Life program text and structure © Vines of Life. The site (this repository) may be shared, copied, and redistributed for non-commercial use, with attribution to Vines of Life and sources left intact. See `LICENSE` for details.

Third-party material quoted here remains the property of its respective rights holders and is used for educational purposes with attribution. If you are a rights holder and would like material revised or removed, please open an issue or contact Vines of Life.
