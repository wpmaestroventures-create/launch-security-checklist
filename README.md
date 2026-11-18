# Launch Security Readiness Checklist
### For vibe-coded, AI-built, and personal websites that collect people's data

Tools like Lovable, Bolt, v0, Cursor, and Copilot make it possible to ship a working site in a weekend. They don't decide whether that site is safe to put in front of real people. This checklist is the review I run before a small site goes live, especially one that takes names, emails, phone numbers, logins, or payments.

It is written for founders, freelancers, and small teams without a security department. Each item is something you can check yourself in under ten minutes.

**Author:** William Pefok, Senior Security Engineer · Cloud Security & Identity
**License:** MIT. Copy it, fork it, use it with clients.

---

## How to use this

1. Copy this file into your project (or open an issue and paste it in).
2. Work through each section and tick the boxes.
3. **Must** items block launch. **Should** items get fixed within 30 days of launch.
4. Fill in the sign-off table at the bottom and keep it with the repo. It becomes your evidence that a review happened.

---

## 1. Secrets and API keys

AI builders often put keys straight into front-end code. Anything in the browser is public.

- [ ] **Must:** No secret keys in front-end code, the Git history, or screenshots. Search the repo for `sk_`, `secret`, `service_role`, `apikey`, and `password`.
- [ ] **Must:** Secrets live in the host's environment variables (Vercel, Netlify, Supabase, and so on), not in `.env` files committed to Git.
- [ ] **Must:** `.env` is listed in `.gitignore`.
- [ ] **Must:** If a secret was ever committed, rotate it. Deleting the file doesn't remove it from history.
- [ ] **Should:** Only public, restricted keys (such as a Supabase anon key or a Stripe publishable key) appear in the browser, and you know what each one can do.
- [ ] **Should:** Turn on GitHub secret scanning and push protection for the repo.

## 2. Accounts and admin access

- [ ] **Must:** MFA is on for every admin account: domain registrar, host, database, GitHub, email provider, payment provider.
- [ ] **Must:** Each person has their own login. No shared admin passwords.
- [ ] **Must:** Default, test, and demo accounts are deleted before launch.
- [ ] **Must:** The domain registrar account is locked and owned by the business, not a freelancer's personal account.
- [ ] **Should:** Write down who has admin access to what, and remove access when someone leaves the project.

## 3. Database and data access (the most common failure)

On AI-built apps the biggest risk is usually a database that answers anyone who asks.

- [ ] **Must:** Row-level security (or the equivalent) is **enabled on every table** that holds user data. In Supabase, check that RLS is on and that no policy is `using (true)` for reads of personal data.
- [ ] **Must:** Test it as an attacker. Log in as User A and try to read or change User B's record by changing an ID in the URL or the API request. You should get nothing back. (This flaw is called IDOR, or broken access control.)
- [ ] **Must:** Logged-out visitors can't list users, signups, messages, or orders through the API.
- [ ] **Must:** Admin pages check the user's role on the server, not just by hiding a button.
- [ ] **Should:** Storage buckets for uploads are private by default, and file links expire.
- [ ] **Should:** The database is not reachable directly from the internet except through the provider's API.

## 4. Authentication

- [ ] **Must:** Use the platform's built-in auth (Supabase Auth, Clerk, Auth0, Firebase, and so on). Don't build your own password storage.
- [ ] **Must:** Email verification is on for new accounts.
- [ ] **Must:** Password reset links expire and work only once.
- [ ] **Should:** Rate limiting or CAPTCHA protects login, signup, and password reset.
- [ ] **Should:** Users can delete their account and their data, and that actually removes it.

## 5. Forms, inputs, and uploads

- [ ] **Must:** Every form validates input on the server, not only in the browser.
- [ ] **Must:** Contact and signup forms have bot protection (CAPTCHA, Turnstile, or a honeypot field) so they can't be used to spam people.
- [ ] **Must:** Uploads are limited by file type and size and are never executed or served from your main domain.
- [ ] **Should:** User-entered text is escaped when shown on a page, so nobody can inject scripts (XSS).

## 6. Personal data and email

If you collect it, you are responsible for it. Collect less.

- [ ] **Must:** List every piece of personal data the site collects, why you need it, and where it is stored. Delete fields you don't need.
- [ ] **Must:** Don't collect sensitive data (government IDs, health details, card numbers) unless the site truly requires it. Use a payment provider so card numbers never touch your site.
- [ ] **Must:** Marketing emails go only to people who opted in, and every email has a working unsubscribe link.
- [ ] **Must:** Your sending domain has SPF, DKIM, and DMARC records set up. This stops others from spoofing your domain and keeps your mail out of spam.
- [ ] **Must:** Form submissions go to a business inbox or CRM, not a personal Gmail that others can't audit.
- [ ] **Should:** Decide how long you keep data and delete old records on a schedule.
- [ ] **Should:** Exports and spreadsheets of user data are not left in shared drives with "anyone with the link" access.

## 7. Hosting and HTTPS

- [ ] **Must:** HTTPS everywhere, with HTTP redirecting to HTTPS.
- [ ] **Must:** Preview and staging deployments are password-protected or don't use real user data.
- [ ] **Should:** Security headers are set: `Content-Security-Policy`, `X-Content-Type-Options: nosniff`, `Referrer-Policy`, and `Strict-Transport-Security`. Check with securityheaders.com.
- [ ] **Should:** Error pages don't show stack traces, database errors, or file paths.

## 8. AI-generated code and dependencies

- [ ] **Must:** A person reads every piece of AI-generated code that touches login, payments, database rules, or personal data before it ships.
- [ ] **Must:** Run `npm audit` (or the equivalent) and fix critical and high findings.
- [ ] **Should:** Turn on Dependabot or Renovate for automatic dependency updates.
- [ ] **Should:** Remove unused packages, test pages, and debug routes the AI tool created.
- [ ] **Should:** If the site uses an AI model, users can't make it reveal system prompts, API keys, or other users' data. Never put secrets in a prompt.

## 9. Logging, monitoring, and backups

- [ ] **Must:** You'll know if the site goes down (uptime monitoring with email or text alerts).
- [ ] **Must:** Database backups are on, and you have restored one at least once to prove it works.
- [ ] **Should:** Failed logins and errors are logged somewhere you actually look.
- [ ] **Should:** Logs don't contain passwords, full card numbers, or reset tokens.

## 10. Privacy and policies

This isn't legal advice. It's the minimum most small sites need.

- [ ] **Must:** A privacy policy explains what you collect, why, who you share it with, and how to request deletion.
- [ ] **Must:** If you have users in the EU or UK (GDPR) or California (CCPA/CPRA), check what those rules require before launch.
- [ ] **Should:** A cookie or tracking notice where required, and analytics configured to collect only what you use.

## 11. If something goes wrong

- [ ] **Must:** Write down who to call and what to do first if data is exposed: rotate keys, take the site offline if needed, and preserve logs.
- [ ] **Should:** Know your notification duties. Many laws require telling affected users within a fixed time after a breach.

---

## Fast check: the 10 most common vibe-coding mistakes

1. Database RLS turned off, or a policy that lets anyone read everything
2. A service-role or secret key shipped in the front-end bundle
3. Admin routes protected only by a hidden button
4. User IDs in URLs that let anyone view another person's data
5. `.env` committed to a public repo
6. Contact forms with no bot protection, used to send spam
7. Preview deployments with real customer data and no password
8. No MFA on the domain registrar or host account
9. Backups never tested
10. Marketing emails sent without opt-in or an unsubscribe link

---

## Launch sign-off

| Item | Result |
|---|---|
| Site / project | |
| Reviewed by | |
| Review date | |
| Must items open | |
| Should items open (with fix-by date) | |
| Decision | Go / Go with accepted risks / No-go |
| Accepted risks and owner | |

---

*Built from hands-on work reviewing enterprise cloud releases and small-business launches. Suggestions and pull requests are welcome.*
