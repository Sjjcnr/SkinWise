# 🌿 SkinWise Advisor — Comprehensive Codebase Analysis & Technical Reference

> **Project Path:** `C:\Users\91638\Downloads\skinwise different versions\SkinWise`  
> **Repository:** [Sjjcnr/SkinWise](https://github.com/Sjjcnr/SkinWise)  
> **Application Type:** AI-Powered Skincare Recommendation & Computer Vision Assessment Platform  
> **License:** ISC  

---

## 1. Executive Summary

**SkinWise Advisor** is a full-stack, AI-driven skincare consultation web application tailored specifically for the Indian skincare ecosystem. It blends client-side Computer Vision (TensorFlow.js BlazeFace) with multimodal generative AI (Google Gemini 2.5 Flash) via a Supabase Edge Function to deliver hyper-personalized skincare routines within specified budget tiers and generate direct Amazon India product purchase links.

### Core Problems Solved
1. **Generic Skincare Advice:** Skincare regimens typically fail to factor in geographic climate, localized allergies, regional brand availability, and strict budget constraints.
2. **Poor Quality User Photos for Diagnostics:** Client-side quality gating verifies user face photos in real-time (lighting, blur, face count, and positioning) before passing images to multimodal LLMs.
3. **Budget & Market Grounding:** Recommendations are grounded in real Indian brands (Minimalist, Dot & Key, Plum, The Derma Co, Cetaphil, etc.) with automatic search links for Indian e-commerce.

---

## 2. Architecture & Technology Stack

```
┌────────────────────────────────────────────────────────────────────────┐
│                        FRONTEND (Vite + React)                         │
│  - React 18.3 / JavaScript (ES2020+) / Vite 5.4                        │
│  - Tailwind CSS 3.4 / shadcn/ui (Radix UI) / Lucide Icons              │
│  - TanStack React Query 5.83 / React Hook Form + Zod                   │
└──────────────────┬────────────────────────────────┬────────────────────┘
                   │                                │
      Client-side CV Validation           Supabase Client (@supabase/supabase-js)
                   │                                │
┌──────────────────▼──────────────────┐   ┌─────────▼────────────────────────────┐
│      TENSORFLOW.JS / BLAZEFACE      │   │           SUPABASE BACKEND           │
│  - BlazeFace Model Inference        │   │  - Auth (Email/Password JWT)         │
│  - Rec. 709 Luminance Gating        │   │  - PostgreSQL Database with RLS      │
│  - Laplacian Blur Variance          │   │  - Edge Functions (Deno Runtime)     │
│  - Geometric Oval Bounds Checks     │   └──────────────────┬───────────────────┘
└─────────────────────────────────────┘                      │
                                             Edge Function: get-recommendations
                                                             │
                              ┌──────────────────────────────┴──────────────────────────────┐
                              ▼                                                             ▼
                Google Gemini 2.5 Flash API                                  Firecrawl Search API
             (Direct Key or Lovable Gateway)                             (Domain Matching & Product Links)
```

### Detailed Tech Stack Breakdown

| Layer | Technologies & Libraries | Key Responsibilities |
|---|---|---|
| **Frontend Framework** | React 18.3.1, JavaScript (ES2020+), Vite 5.4.19 | Component hierarchy, client routing, fast HMR bundling |
| **Styling & Design System** | Tailwind CSS 3.4.17, `tailwindcss-animate`, `next-themes`, shadcn/ui | Warm earthy aesthetic (`skinwise-sage`, `skinwise-terracotta`, `skinwise-bark`), dark/light mode toggle |
| **Primitives & UI Kit** | Radix UI (20+ primitives: Dialog, Accordion, Tooltip, Dropdown, Slider, etc.), Sonner | Headless, accessible UI components |
| **Forms & State** | React Hook Form 7.61, Zod 3.25, TanStack Query 5.83 | Schema validation, wizard state management, server cache invalidation |
| **Computer Vision** | `@tensorflow/tfjs` 4.22, `@tensorflow-models/blazeface` 0.1.0 | In-browser face detection, quality analysis, webcam stream capture |
| **Backend & Database** | Supabase (PostgreSQL 14.1, GoTrue Auth, Row Level Security) | User accounts, profiles, assessments history, favorites, feedback |
| **Edge Compute** | Supabase Edge Functions (Deno 1.25+ / JavaScript) | Secure API key proxy, AI prompt orchestration, e-commerce web search |
| **AI Engine** | Google Gemini 2.5 Flash (`gemini-2.5-flash`) | Multimodal skin analysis (photo + text profile) and structured JSON output |
| **Search & Enrichment**| Firecrawl API (`api.firecrawl.dev/v1/search`) | Real-time Indian e-commerce link discovery with Amazon fallback |

---

## 3. Directory & File Structure

```
SkinWise/
├── .env                          # Supabase & API keys
├── components.json               # shadcn/ui configuration (configured for JS)
├── index.html                    # Root HTML document
├── jsconfig.json                 # JavaScript compiler & path alias configuration (@/ -> src/)
├── package.json                  # Dependencies & npm scripts
├── tailwind.config.js            # Custom themes, color palette, animations
├── vite.config.js                # Vite build plugins & path aliases (@/ -> src/)
│
├── supabase/
│   ├── config.toml               # Supabase project configuration
│   ├── migrations/               # SQL migrations (heartbeat, delete policies)
│   └── functions/
│       └── get-recommendations/
│           └── index.js          # Deno edge function (Gemini 2.5 + Firecrawl)
│
└── src/
    ├── App.jsx                   # Main router with protected routes & providers
    ├── index.css                 # Global CSS variables, custom utilities & fonts
    ├── main.jsx                  # React DOM root entry point
    │
    ├── contexts/
    │   └── AuthContext.jsx       # Supabase Auth provider (session, user, auth state)
    │
    ├── hooks/
    │   ├── useAssessment.js      # Multi-step quiz state, validation & navigation
    │   ├── useFaceDetection.js   # TensorFlow.js BlazeFace + image quality diagnostics
    │   ├── use-mobile.jsx        # Responsive screen breakpoint detector
    │   └── use-toast.js          # Toast dispatch hook
    │
    ├── integrations/
    │   └── supabase/
    │       ├── client.js         # Supabase client instantiation
    │       └── types.js          # JavaScript Supabase database types
    │
    ├── types/
    │   └── skincare.js           # Domain constants: Assessment, Skin Types, Concerns, Budgets
    │
    ├── pages/
    │   ├── Index.jsx             # Landing page with hero, features, CTA
    │   ├── Auth.jsx              # Login & registration tabs with Supabase auth
    │   ├── Quiz.jsx              # 6-step assessment wizard
    │   ├── Results.jsx           # Product recommendation cards & routine display
    │   ├── History.jsx           # Assessment log with expandable items & 8s undo delete
    │   ├── Favorites.jsx         # Bookmarked products grid with filtering
    │   ├── Profile.jsx           # User settings, password update & account stats
    │   ├── FaceCapturePage.jsx   # Dedicated full-page webcam scanner
    │   └── NotFound.jsx          # 404 page
    │
    └── components/
        ├── BrandLogo.jsx         # SVG Leaf + Logotype branding
        ├── FaceCapture.jsx       # Webcam stream, oval guide overlay, canvas processing
        ├── FavoriteButton.jsx    # Optimistic bookmarking with Supabase persistence
        ├── ProductFeedback.jsx   # Rating (1-5 stars) and review notes per product
        ├── ProtectedRoute.jsx    # Auth guard redirecting to /auth
        ├── ThemeProvider.jsx     # Theme provider wrapper (next-themes)
        ├── ThemeToggle.jsx       # Light / Dark mode toggle switch
        ├── quiz/                 # Step-by-step assessment components
        │   ├── ProgressIndicator.jsx # 6-step visual timeline
        │   ├── StepBasicInfo.jsx     # Age range & gender selector
        │   ├── StepSkinType.jsx      # Oily, dry, combination, sensitive, normal
        │   ├── StepConcerns.jsx      # Multi-select skin concerns
        │   ├── StepLifestyle.jsx     # Climate conditions & allergen flags
        │   ├── StepPreferences.jsx   # Budget tier (₹500 to ₹3000+) & goals
        │   └── StepFacePhoto.jsx     # Photo capture or file upload step
        └── ui/                       # shadcn/ui primitives (Button, Card, Dialog...)
```

---

## 4. Key Functional Modules & Technical Mechanics

### 4.1. Smart 6-Step Skin Assessment Quiz (`src/hooks/useAssessment.js`)
The assessment is managed via a dedicated hook maintaining step index and immutable updates:
1. **Step 1: Basic Info** — Age ranges (`under18`, `18-24`, `25-34`, `35-44`, `45-54`, `55+`) and gender.
2. **Step 2: Skin Type** — Single choice with detailed descriptions (Oily, Dry, Combination, Sensitive, Normal).
3. **Step 3: Skin Concerns** — Multi-select (Acne, Pigmentation, Dryness, Aging, Redness, Dullness, Large Pores, Excess Oil, Texture, Dark Circles).
4. **Step 4: Lifestyle & Environment** — Climate (Tropical, Dry/Arid, Temperate, Cold, Varied) and common allergens (Fragrance, Parabens, Sulfates, Alcohol, Retinol, Vitamin C, Salicylic Acid, Niacinamide).
5. **Step 5: Preferences & Budget** —
   - `budget`: Under ₹500 INR
   - `mid`: ₹500–₹1,500 INR
   - `premium`: ₹1,500–₹3,000 INR
   - `luxury`: ₹3,000+ INR
6. **Step 6: Face Photo (Optional)** — Real-time camera capture with face validation or file upload.

### 4.2. In-Browser Computer Vision Diagnostics (`src/hooks/useFaceDetection.js`)
Before submitting a face photo to the backend, the client performs strict on-device computer vision checks using HTML5 Canvas and TensorFlow.js:
- **Resolution Check:** Validates width $\ge 800\text{px}$.
- **Luminance Calculation (Rec. 709 Standard):**
  $$\text{Luminance} = 0.2126 \cdot R + 0.7152 \cdot G + 0.0722 \cdot B$$
  Rejects under-exposed images ($< 50$) and over-exposed images ($> 220$).
- **Laplacian Blur Detection:**
  Computes grayscale image data, applies a $3 \times 3$ Laplacian kernel:
  $$\begin{bmatrix} 0 & 1 & 0 \\ 1 & -4 & 1 \\ 0 & 1 & 0 \end{bmatrix}$$
  Calculates variance $\sigma^2 = \frac{1}{N}\sum (\text{lap} - \mu)^2$. Rejects blurry images where $\sigma^2 < 120$.
- **TensorFlow.js BlazeFace Detection:**
  - Enforces `estimateFaces` returns exactly 1 face.
  - Checks prediction confidence score $\ge 0.75$.
  - Executes **`isBboxInsideOval`**: verifies that the bounding box center lies within the UI framing oval and covers between 15% and 95% of the oval's area.

### 4.3. Supabase Edge Function (`supabase/functions/get-recommendations/index.js`)
- **Execution Environment:** Hosted on Supabase (using Deno runtime).
- **Authentication:** Validates incoming user JWT against `supabaseClient.auth.getUser(token)`.
- **Multimodal Prompt Engineering:**
  - Builds dynamic constraints based on budget tiers, climate, and user allergies ("Do NOT recommend products containing...").
  - Appends base64-encoded image parts (`inlineData`) for Gemini 2.5 Flash if a photo was provided.
  - Enforces strict JSON output schema containing product category, brand, key ingredients, price range, why suitable, and usage instructions.
- **Product Link Generation:**
  - Uses an automatic Amazon India query generator (`https://www.amazon.in/s?k=${encodeURIComponent(brand + ' ' + name)}`) to allow users to directly look up and buy the recommended products.
  - *(Note: A Firecrawl search enrichment block exists in the code as an optional branch, but runs the Amazon fallback when `FIRECRAWL_API_KEY` is not set).*

### 4.4. Assessment History with 8-Second Undo (`src/pages/History.jsx`)
- Fetches all assessments joined with corresponding recommendation sets.
- Implements an **Optimistic Undo Deletion** pattern:
  - When user triggers delete, item is immediately removed from the active UI list.
  - A toast with an 8-second countdown timer (`Countdown` component) and an "Undo" action appears.
  - If the timer expires without cancellation, the permanent Supabase delete query executes.

### 4.5. Favorites & Interactive Feedback Loop
- **`FavoriteButton.jsx`:** Toggles bookmarking directly in the `favorites` table with optimistic UI updates.
- **`ProductFeedback.jsx`:** Allows users to rate each recommended product (1–5 stars), specify if they have tried it, and record written notes in the `product_feedback` table.

---

## 5. Database Schema (Supabase / PostgreSQL)

```sql
-- 1. Profiles Table
CREATE TABLE profiles (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL UNIQUE,
    full_name TEXT,
    email TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- 2. Skin Assessments Table
CREATE TABLE skin_assessments (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    age_range TEXT NOT NULL,
    gender TEXT,
    skin_type TEXT NOT NULL,
    skin_concerns TEXT[] NOT NULL,
    climate TEXT,
    allergies TEXT[],
    skin_goals TEXT,
    budget_range TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 3. Recommendations Table
CREATE TABLE recommendations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    assessment_id UUID REFERENCES skin_assessments(id) ON DELETE CASCADE,
    user_id UUID NOT NULL,
    products JSONB NOT NULL,
    ai_summary TEXT,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 4. Favorites Table
CREATE TABLE favorites (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL,
    product_name TEXT NOT NULL,
    product_brand TEXT,
    product_data JSONB NOT NULL,
    created_at TIMESTAMPTZ DEFAULT NOW()
);

-- 5. Product Feedback Table
CREATE TABLE product_feedback (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    recommendation_id UUID REFERENCES recommendations(id) ON DELETE CASCADE,
    user_id UUID NOT NULL,
    product_name TEXT NOT NULL,
    rating INTEGER NOT NULL CHECK (rating >= 1 AND rating <= 5),
    feedback TEXT,
    tried_product BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMPTZ DEFAULT NOW(),
    updated_at TIMESTAMPTZ DEFAULT NOW()
);
```

---

## 6. Environment Configuration

| Variable | Scope | Purpose |
|---|---|---|
| `VITE_SUPABASE_URL` | Client (`.env`) | Supabase project URL |
| `VITE_SUPABASE_PUBLISHABLE_KEY` | Client (`.env`) | Supabase public anonymous / publishable key |
| `GEMINI_API_KEY` | Edge Function Secret | Google Gemini API key for `gemini-2.5-flash` |
| `LOVABLE_API_KEY` | Edge Function Secret (Optional) | Gateway fallback for AI chat completions |
| `FIRECRAWL_API_KEY` | Edge Function Secret (Optional) | Firecrawl web search API key for shopping links |

---

## 7. Resume-Ready Highlights & Technical Bullet Points

- **AI-Powered Skincare Advisor:** Built a full-stack automated skincare diagnosis platform serving the Indian retail skincare market using **React 18, JavaScript (ES2020+), Tailwind CSS, and Supabase**.
- **Client-Side Computer Vision Pipeline:** Engineered an on-device image validation suite using **TensorFlow.js BlazeFace**, calculating **Rec. 709 luminance** and **Laplacian blur variance** to gate camera photo uploads prior to LLM processing.
- **Multimodal LLM Orchestration:** Developed a Deno-based **Supabase Edge Function** invoking **Google Gemini 2.5 Flash** with multimodal base64 image parsing, dynamic allergy exclusion filters, and strictly budgeted price-tier enforcement.
- **E-Commerce Search Enrichment:** Integrated the **Firecrawl Search API** to dynamically discover verified product purchasing links across major Indian e-commerce portals (**Amazon, Nykaa, Flipkart, Purplle**).
- **Resilient UX Patterns:** Implemented an **optimistic UI deletion system with an 8-second countdown timer undo mechanism**, accessible dialog primitives with **Radix UI**, and dark/light system theme detection.
