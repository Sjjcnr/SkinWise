# SkinWise Advisor

An intelligent skin assessment tool that curates personalized product recommendations tailored to your unique skin needs. Built with a modern React frontend and a Supabase backend to deliver AI-driven insights directly to you.

## Features

- **Personalized Assessment:** Analyze skin type, concerns, climate, and more.
- **AI-Powered Recommendations:** Utilizes the Gemini API for precise, context-aware suggestions.
- **Modern UI:** Clean, glassmorphic interface that's fully responsive.
- **Real-Time Database:** Backend managed securely via Supabase.

## Setup

1. **Clone the repository:**
   ```bash
   git clone <repository_url>
   ```

2. **Install dependencies:**
   Ensure you have Node.js and Bun installed.
   ```bash
   npm install
   # or
   bun install
   ```

3. **Environment Setup:**
   Create a `.env` file in the root based on your Supabase and Gemini settings:
   ```env
   VITE_SUPABASE_URL=your_supabase_url
   VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
   ```

4. **Run the development server:**
   ```bash
   npm run dev
   # or
   bun dev
   ```

## Tech Stack

- **Frontend:** Vite, React, Tailwind CSS, TypeScript
- **Backend:** Connected remotely to Supabase (Database, Auth)
