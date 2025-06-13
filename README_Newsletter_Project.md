
# 📬 Newsletter Signup App

A lightweight, production-ready newsletter signup application built with **Next.js**, **Vercel Serverless Functions**, **Google Sheets API**, and **Tailwind CSS**.

## 🚀 Features

- React form with real-time validation
- Serverless function to store signups in Google Sheets
- Tailwind CSS for responsive design
- Environment variable support for credentials
- Deployable to Vercel or any Node.js-friendly platform

---

## 📁 Project Structure

```
newsletter-app/
├── api/
│   └── subscribe.js           # Vercel serverless function
├── components/
│   └── NewsletterForm.jsx     # React form component
├── pages/
│   ├── _app.js                # Global CSS import
│   └── index.js               # Main page
├── styles/
│   └── globals.css            # Tailwind CSS styles
├── .env.local.example         # Sample environment config
├── tailwind.config.js         # Tailwind config
├── postcss.config.js          # PostCSS config
├── README.md                  # This file
└── package.json               # Dependencies and scripts
```

---

## ⚙️ Environment Setup

Copy the `.env.local.example` and update with your Google Sheets credentials:

```bash
cp .env.local.example .env.local
```

```env
GOOGLE_SERVICE_ACCOUNT_EMAIL=your-service-account@your-project.iam.gserviceaccount.com
GOOGLE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\nABC123...\n-----END PRIVATE KEY-----\n"
GOOGLE_SHEET_ID=your_google_sheet_id
```

> Make sure to replace actual line breaks in the private key with `\n`

---

## 🧪 Running Locally

```bash
npm install
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000)

---

## 🌐 Deployment

Deploy to [Vercel](https://vercel.com/) in minutes:

1. Import GitHub repo into Vercel
2. Set environment variables under project settings
3. Click **Deploy**

---

## ✅ Checklist

- [x] Tailwind CSS installed
- [x] Google Sheets API configured
- [x] Environment variables set
- [x] Vercel function ready
- [x] React frontend complete

---

## 🧩 Optional Enhancements

- Add reCAPTCHA to form
- Send confirmation email with SendGrid
- Export submissions to CSV
- Display a success message on the page

---

## 📄 License

MIT License – Free to use and customize.
