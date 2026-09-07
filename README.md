# Assessment deliverables

## landing-page/
Single-file static site (index.html). Fonts load from Google Fonts; nothing else is external.
Deploy on Netlify: drag the `landing-page` folder onto app.netlify.com/drop, or `netlify deploy --dir=landing-page --prod`.
To connect the form to a CRM or webhook, set the `ENDPOINT` constant near the bottom of index.html; the page POSTs `{name, email, phone, event}` as JSON.
To move the event slot, edit `SLOT` (weekday 0-6 and UTC hour) in the same script.

## dashboard/
Single-file static site (index.html) with the anonymised dataset embedded (no names, phones, emails or free-text answers).
Deploy the same way: drag the `dashboard` folder onto Netlify Drop, or `netlify deploy --dir=dashboard --prod`.
If you want both on one site, put `dashboard/` inside `landing-page/` and it will be served at `/dashboard/`.

## image-ads/
25 PNGs at 1080x1080, named ad-01 to ad-25. Share the folder as is.

## ai-system-answer.md
Draft answer for the "most complex AI system" question; fill the bracketed numbers before submitting.
