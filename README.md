
# Garden Growth Companion

A **Progressive Web App (PWA)** designed to help gardeners grow plants from seeds to maturity.  
The app offers plant care guides, personalised plant logs, watering reminders, and a friendly gardening community.

---

## 🚀 Features (MVP)
- **User Authentication** – Sign up with email/password or Google.
- **Plant Library** – Searchable database with images, descriptions, and care tips.
- **Personal Logs** – Record planting dates, notes, and photos for each plant.
- **Reminders** – Push notifications for watering, fertilising, and planting schedules.
- **Offline Mode** – Installable app with offline access to plant library.

---

## 🛠 Tech Stack
- **Frontend:** Next.js, React, Tailwind CSS, shadcn/ui
- **Backend:** Supabase (PostgreSQL, Auth, Storage)
- **Notifications:** Web Push API or OneSignal
- **Deployment:** Vercel
- **Analytics:** Plausible (GDPR-friendly)

---

## 📦 Getting Started

### Prerequisites
- Node.js (v18+)
- pnpm or npm
- Supabase project (with Auth, Database, Storage enabled)

### Installation
```bash
# Clone repository
git clone https://github.com/yourusername/garden-growth-companion.git
cd garden-growth-companion

# Install dependencies
pnpm install

# Set up environment variables
cp .env.example .env.local
# Add your Supabase keys and other credentials

# Run development server
pnpm dev
```

---

## 📂 Project Structure
```
/app               # Next.js App Router pages & layouts
/components        # UI components
/lib               # Utilities & services (Supabase, notifications, validation)
/public            # Static files (manifest.json, icons)
/styles            # Tailwind styles
/workers           # Service workers
```

---

## 🌱 Roadmap
1. MVP launch with authentication, plant library, logs, and reminders.
2. Add weather-based reminder adjustments.
3. Introduce community features (photo sharing, discussions).
4. Gamification with badges for milestones.
5. Expand plant database & multilingual support.

---

## 🤝 Contributing
Pull requests are welcome!  
Please read [`agent.md`](./agent.md) for technical guidance and project rules.

---

## 📜 License
MIT License — See [`LICENSE`](./LICENSE) for details.
