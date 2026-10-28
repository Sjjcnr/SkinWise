# SkinWise Advisor — Complete Engineering Course & Technical Viva Guide

> **Target Audience:** Engineering Students, Junior Developers, and Technical Interviewees  
> **Course Objective:** Master the inner workings, design choices, trade-offs, and failure modes of the **SkinWise** codebase to defend every architectural and algorithmic decision in a technical interview or college viva.

---

## 1. Project Overview

**SkinWise** is an AI-powered skincare consultation and computer vision platform designed specifically for the Indian skincare ecosystem. The application guides users through a multi-step dermatological survey, performs real-time client-side computer vision quality gating on an optional face photo using TensorFlow.js and custom image processing heuristics, and orchestrates multimodal generative AI (Google Gemini 2.5 Flash) via a secure backend Edge Function to generate clinically informed product recommendations strictly matched to Indian budgets and retail markets (Amazon India, Nykaa, Flipkart).

### Full Technology Stack & Architectural Roles

| Technology | Version / Spec | Role in SkinWise |
|---|---|---|
| **React** | `^18.3.1` | Client-side component tree, virtual DOM rendering, and user interface state management. |
| **JavaScript (ES2020+)** | Modern ES | Component hierarchy, hooks-based state management, dynamic JSON handling, and fast browser execution. |
| **Vite** | `^5.4.19` | Next-generation frontend build tooling, ES module local dev server, and production bundle optimization. |
| **Tailwind CSS** | `^3.4.17` | Utility-first CSS styling engine configured with an earthy spa-inspired color palette (`cream`, `terracotta`, `sage`, `moss`, `bark`). |
| **shadcn/ui & Radix UI** | Multi-package | Unstyled, fully accessible, headless UI primitives (dialogs, accordions, tooltips, cards, buttons) styled with Tailwind CSS. |
| **TanStack React Query** | `^5.83.0` | Asynchronous state container, caching, and server state synchronization on the frontend. |
| **React Router DOM** | `^6.30.1` | Declarative client-side routing, route protection guards, URL parameters, and browser history synchronization. |
| **TensorFlow.js (`@tensorflow/tfjs`)** | `^4.22.0` | In-browser machine learning runtime executing deep learning models via WebGL without backend round-trips. |
| **BlazeFace (`@tensorflow-models/blazeface`)** | `^0.1.0` | Lightweight sub-millisecond neural network for face detection, bounding box extraction, and face alignment. |
| **Supabase (PostgreSQL & GoTrue)** | PostgreSQL 14.1 / `@supabase/supabase-js ^2.89.0` | Managed cloud relational database, Row Level Security (RLS) data protection, and JWT-based user authentication. |
| **Supabase Edge Functions** | Deno `1.25+` | Serverless, edge-deployed Deno/JavaScript runtime hosting backend endpoints without exposing sensitive API keys to the browser. |
| **Google Gemini 2.5 Flash** | `gemini-2.5-flash` via REST API | Multimodal Large Language Model processing structured skin profiles and raw image bytes to output dermatological regimens. |
| **Firecrawl Search API** | `api.firecrawl.dev/v1/search` | Web scraping and search engine API resolving live Indian e-commerce product links (Amazon.in, Nykaa) with automated fallback. |
| **Zod** | `^3.25.76` | Schema declaration and runtime data validation for form inputs and user credentials. |

---

### Architecture & System Topology

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                                   CLIENT LAYER (Browser)                                │
│                                                                                        │
│   React 18 Component Tree ──> React Router (Protected Routes) ──> Contexts (Auth/Theme)│
│               │                                                                        │
│               ▼                                                                        │
│   Client-side CV Validation Pipeline                                                   │
│   ├── Canvas 2D Context (Pixel Extraction)                                             │
│   ├── Rec. 709 Weighted Luminance Check (Lighting verification)                       │
│   ├── Discrete 3x3 Laplacian Kernel (Blur detection via variance)                      │
│   └── TensorFlow.js BlazeFace Model (Face detection & oval bounding alignment)         │
└───────────────────────────┬────────────────────────────────────────────┬───────────────┘
                            │                                            │
               Direct Table Queries (RLS)                   Edge Function Invoke (JWT)
                            │                                            │
                            ▼                                            ▼
┌───────────────────────────────────────────────┐  ┌─────────────────────────────────────┐
│             SUPABASE BACKEND (Postgres)       │  │        SUPABASE EDGE FUNCTION       │
│                                               │  │       (get-recommendations)         │
│  - auth.users (GoTrue Auth & Session JWTs)    │  │                                     │
│  - public.skin_assessments                    │  │  1. Manual JWT Verification        │
│  - public.recommendations                     │  │  2. Prompt & Image Packaging       │
│  - public.favorites                           │  │  3. Multimodal Inference Call       │
│  - public.product_feedback                    │  │  4. E-commerce Link Resolution      │
│  - public.profiles                            │  └───────────────┬─────────────────────┘
│  - Row Level Security (auth.uid() = user_id)  │                  │
└───────────────────────────────────────────────┘                  │
                                                                   │
                              ┌────────────────────────────────────┴────────────────────────────────────┐
                              ▼                                                                         ▼
            ┌───────────────────────────────────┐                     ┌───────────────────────────────────┐
            │       GOOGLE GEMINI 2.5 FLASH     │                     │         FIRECRAWL SEARCH API      │
            │   Multimodal Skin Analysis &      │                     │   Real-time Indian e-commerce     │
            │   Structured JSON Routine Output  │                     │   product link discovery          │
            └───────────────────────────────────┘                     └───────────────────────────────────┘
```

---

## 2. Folder & File Map

```
SkinWise/
├── .env                                       # Local environment variables (Supabase URL, Anon Key)
├── .env.local                                 # Local-only environment variable overrides (ignored by Git)
├── .env.production.local                      # Production environment configurations
├── components.json                            # shadcn/ui configuration defining path aliases and CSS variables
├── eslint.config.js                           # ESLint configuration specifying rules for React Hooks and Refresh
├── index.html                                 # Single-page application HTML entry shell with fonts and root div
├── package.json                               # Node.js project manifest with dependencies, scripts, and engine specs
├── postcss.config.js                          # PostCSS plugin configuration for Tailwind CSS and Autoprefixer
├── README.md                                  # Developer setup guide and high-level product documentation
├── tailwind.config.js                         # Tailwind CSS configuration declaring custom color tokens and animations
├── jsconfig.json                             # JavaScript configuration defining path aliases (@/ -> src/)
├── vercel.json                                # Vercel deployment configuration routing all paths to index.html
├── vite.config.js                             # Vite configuration with React SWC plugin and @ path alias
│
├── public/
│   ├── favicon.svg                            # Application favicon vector asset
│   ├── placeholder.svg                        # Default fallback placeholder image for missing media
│   └── robots.txt                             # Search engine crawler directives
│
├── src/
│   ├── App.css                                # Global CSS overrides
│   ├── App.jsx                                # Main application root with providers and React Router route table
│   ├── index.css                              # Design system entry file defining HSL variables, dark mode, & font imports
│   ├── main.jsx                               # Application runtime entry point rendering React DOM root
│   │
│   ├── components/
│   │   ├── BrandLogo.jsx                      # Custom SVG brand logo with Great Vibes & Bowlby One SC typography
│   │   ├── FaceCapture.jsx                    # Camera feed manager with SVG guide mask, capture triggers, and consent
│   │   ├── FavoriteButton.jsx                 # Heart toggle button persisting product favorites in Supabase
│   │   ├── NavLink.jsx                        # React Router navigation link wrapper supporting active class states
│   │   ├── ProductFeedback.jsx                # Star rating and review submission dialog for recommended products
│   │   ├── ProtectedRoute.jsx                 # Route guard redirecting unauthenticated visitors to /auth
│   │   ├── ThemeProvider.jsx                  # Next-themes wrapper providing dark/light mode context
│   │   ├── ThemeToggle.jsx                    # Header button toggling dark and light mode themes
│   │   │
│   │   ├── quiz/
│   │   │   ├── ProgressIndicator.jsx          # Multi-step progress bar showing numbered bubbles and completion checkmarks
│   │   │   ├── StepBasicInfo.jsx              # Quiz Step 1: Age range and gender selector
│   │   │   ├── StepSkinType.jsx               # Quiz Step 2: Skin type cards (Oily, Dry, Combination, Sensitive, Normal)
│   │   │   ├── StepConcerns.jsx               # Quiz Step 3: Multi-select skin concerns (Acne, Pigmentation, Aging, etc.)
│   │   │   ├── StepLifestyle.jsx              # Quiz Step 4: Climate selector, allergy checkboxes, and free-form goals
│   │   │   ├── StepPreferences.jsx            # Quiz Step 5: Indian Rupee budget tier selector (Budget, Mid, Premium, Luxury)
│   │   │   └── StepFacePhoto.jsx              # Quiz Step 6: Optional camera capture integration step
│   │   │
│   │   └── ui/                                # 40+ Headless shadcn/ui components (Button, Dialog, Card, Input, etc.)
│   │
│   ├── contexts/
│   │   └── AuthContext.jsx                    # Centralized authentication provider managing Supabase GoTrue user sessions
│   │
│   ├── hooks/
│   │   ├── use-mobile.jsx                     # Responsive hook querying window.matchMedia for mobile viewports
│   │   ├── use-toast.js                       # Toast notification dispatcher and state reducer
│   │   ├── useAssessment.js                   # Custom hook managing quiz state, navigation step logic, and step validity
│   │   └── useFaceDetection.js                # Computer vision engine calculating luminance, blur, and BlazeFace inference
│   │
│   ├── integrations/
│   │   └── supabase/
│   │       ├── client.js                      # Instantiated Supabase client with localStorage session persistence
│   │       └── types.js                       # Auto-generated TypeScript database types matching Supabase tables
│   │
│   ├── lib/
│   │   └── utils.js                           # Tailwind CSS class merging utility combining clsx and tailwind-merge
│   │
│   ├── pages/
│   │   ├── Auth.jsx                           # User sign-in and registration page with Zod client-side validation
│   │   ├── FaceCapturePage.jsx                # Dedicated stand-alone camera capture diagnostic page
│   │   ├── Favorites.jsx                      # Bookmarked products dashboard with remove and purchase actions
│   │   ├── History.jsx                        # Assessment history timeline with 8-second undo countdown deletion
│   │   ├── Index.jsx                          # Landing page with hero section, statistics, and call-to-action buttons
│   │   ├── NotFound.jsx                       # Catch-all 404 error page
│   │   ├── Profile.jsx                        # User profile settings form supporting display name editing
│   │   ├── Quiz.jsx                           # Main 6-step assessment wizard orchestrating step components
│   │   └── Results.jsx                        # AI routine presentation page invoking Edge Function and displaying products
│   │
│   └── types/
│       └── skincare.js                        # Domain interfaces (SkinAssessment, Product, Recommendation) & constants
│
└── supabase/
    ├── config.toml                            # Supabase CLI project configuration and Edge Function settings
    ├── functions/
    │   └── get-recommendations/
    │       └── index.js                       # Deno Edge Function calling Gemini 2.5 Flash and Firecrawl API
    └── migrations/
        ├── 20260712_create_heartbeat_table.sql # Keepalive table preventing auto-pause on Supabase Free Tier
        └── 20260719111000_add_delete_policies.sql # Postgres RLS policies allowing authenticated users to delete assessments
```

---

## 3. File-by-File Deep Dive

This section dissects every meaningful source file across the project. For each file, we unpack its architectural role, walk through its verbatim code, demystify beginner-level concepts, critique design alternatives, and map its dependency graph.

---

### `vite.config.js`

* **What it does:** Configures the Vite development server and build toolchain. It defines the React Fast Refresh compiler plugin, sets network host bindings, and maps the `@` path alias to the `./src` directory.

* **Full walkthrough:**
```javascript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react-swc";
import path from "path";

export default defineConfig({
  server: {
    host: "127.0.0.1",  // Changed from "::" to explicit IPv4
    port: 8080,
    strictPort: true,   // Add this
    allowedHosts: [
      'localhost',
      '127.0.0.1',
      '.ngrok-free.app',
      '.ngrok.io'
    ]
  },
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```
The file exports a configuration object wrapped in `defineConfig()`, which provides IDE autocomplete for Vite options.
1. **Network configuration (`server`):** Forces the local development server to bind explicitly to `127.0.0.1` on port `8080`. Setting `strictPort: true` ensures Vite fails immediately with an error if port 8080 is already occupied rather than silently picking an arbitrary alternative port like 8081. The `allowedHosts` array permits reverse-proxy tunneling via services like ngrok, allowing developers to test mobile camera feeds on physical smartphones over HTTPS.
2. **Plugins (`plugins: [react()]`):** Utilizes `@vitejs/plugin-react-swc`, which leverages SWC (Speedy Web Compiler written in Rust) rather than Babel to perform JSX transformation and Hot Module Replacement (HMR) at near-instantaneous speeds.
3. **Module Resolution (`resolve.alias`):** Binds the symbol `@` to the physical absolute filesystem path of `./src`. This eliminates fragile relative import chains such as `../../components/ui/button`.

* **Concepts a beginner might not know:**
  * **Hot Module Replacement (HMR):** A dev-server mechanism that swaps, adds, or removes code modules in a running application without triggering a full page refresh, preserving current UI component state (e.g., preserving half-completed quiz fields while editing styles).
  * **SWC (Speedy Web Compiler):** An extensible compiler written in Rust that transforms modern JSX and JavaScript into browser-compatible JavaScript up to 20x faster than traditional JavaScript-based compilers like Babel.
  * **Path Aliases:** Shortcuts defined in the bundler config and JavaScript configuration (jsconfig.json) mapping a shorthand character (like `@`) to a specific directory so imports stay clean and resilient to folder refactoring.
  * **Reverse Proxy / Tunneling (`ngrok`):** A utility that exposes a local development server running on your computer to a public HTTPS URL on the internet, which is critical for mobile testing because browser camera APIs (`navigator.mediaDevices.getUserMedia`) are blocked on insecure HTTP connections.

* **Why this approach, not an alternative:**
  * *SWC vs. Babel:* Vite supports both `@vitejs/plugin-react` (Babel) and `@vitejs/plugin-react-swc` (SWC). SWC was chosen for superior compile speed. However, if custom Babel AST transformation plugins were required, SWC would not support them.
  * *Strict Port 8080 vs. Dynamic Port Allocation:* Default Vite behavior increments the port (`8080` -> `8081`) if occupied. Setting `strictPort: true` is crucial when external webhook redirects (like Supabase Auth redirect URLs) or ngrok tunnels expect a deterministic port.

* **Connects to:**
  * **Imported by:** Vite build CLI when running `npm run dev` or `npm run build`.
  * **Interacts with:** `jsconfig.json` (which mirrors the `@/*` path mapping in its `paths` compiler option so the IDE resolves imports seamlessly).

---

### `src/main.jsx`

* **What it does:** The JavaScript entry point of the client application. It grabs the root DOM element from `index.html` and mounts the React virtual DOM tree using React 18's Concurrent Mode renderer.

* **Full walkthrough:**
```javascript
import { createRoot } from "react-dom/client";
import App from "./App.jsx";
import "./index.css";

createRoot(document.getElementById("root")).render(<App />);
```
The file executes three concise actions:
1. Imports `createRoot` from the `react-dom/client` package.
2. Imports the top-level application root component (`App.jsx`) and the global CSS stylesheet (`index.css`), which injects the Tailwind base utilities and custom font imports.
3. Finds the `<div id="root"></div>` element in the document and calls `.render(<App />)`. Mounts the application to the root DOM element from `index.html`.

* **Concepts a beginner might not know:**
  * **DOM Root (Mounting):** The physical HTML node on the webpage where React takes control and injects all dynamically generated virtual DOM elements.
    * **React 18 `createRoot` vs legacy `ReactDOM.render`:** React 18 replaced the older `render()` API with `createRoot()` to activate concurrent rendering capabilities, such as automatic batching of state updates and non-blocking transitions.

* **Why this approach, not an alternative:**
  * *Standard Single-Page Application (SPA) mounting:* This is the idiomatic entry pattern for modern React SPAs. An alternative would be Server-Side Rendering (SSR) via Next.js or Remix. For a client-side computer vision app running BlazeFace on WebGL canvas, an SPA is ideal because client hardware access and in-memory model execution are central to the user experience.

* **Connects to:**
  * **Imports from:** `react-dom/client`, [App.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/App.jsx), and [index.css](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/index.css).
  * **Interacts with:** `index.html` (specifically target element `<div id="root"></div>`).

---

### `src/App.jsx`

* **What it does:** The primary architectural orchestrator for the frontend. It establishes the global Provider Tree (Query Client, Theme, Authentication, Tooltip, and Toasters) and declares the complete client-side routing table.

* **Full walkthrough:**
```javascript
import { Toaster } from "@/components/ui/toaster";
import { Toaster as Sonner } from "@/components/ui/sonner";
import { TooltipProvider } from "@/components/ui/tooltip";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { BrowserRouter, Routes, Route } from "react-router-dom";
import { AuthProvider } from "@/contexts/AuthContext";
import { ThemeProvider } from "@/components/ThemeProvider";
import ProtectedRoute from "@/components/ProtectedRoute";
import Index from "./pages/Index";
import Auth from "./pages/Auth";
import Quiz from "./pages/Quiz";
import Results from "./pages/Results";
import History from "./pages/History";
import Profile from "./pages/Profile";
import Favorites from "./pages/Favorites";

import FaceCapturePage from "./pages/FaceCapturePage";
import NotFound from "./pages/NotFound";

const queryClient = new QueryClient();

const App = () => (
  <QueryClientProvider client={queryClient}>
    <ThemeProvider
      attribute="class"
      defaultTheme="system"
      enableSystem
      disableTransitionOnChange
    >
      <AuthProvider>
        <TooltipProvider>
          <Toaster />
          <Sonner />
          <BrowserRouter>
            <Routes>
              {/* Public routes */}
              <Route path="/" element={<Index />} />
              <Route path="/auth" element={<Auth />} />

              {/* Protected routes — redirect to /auth when not signed in */}
              <Route path="/quiz" element={<ProtectedRoute><Quiz /></ProtectedRoute>} />
              <Route path="/results/:assessmentId" element={<ProtectedRoute><Results /></ProtectedRoute>} />
              <Route path="/history" element={<ProtectedRoute><History /></ProtectedRoute>} />
              <Route path="/profile" element={<ProtectedRoute><Profile /></ProtectedRoute>} />
              <Route path="/favorites" element={<ProtectedRoute><Favorites /></ProtectedRoute>} />
              <Route path="/face-capture" element={<ProtectedRoute><FaceCapturePage /></ProtectedRoute>} />

              <Route path="*" element={<NotFound />} />
            </Routes>
          </BrowserRouter>
        </TooltipProvider>
      </AuthProvider>
    </ThemeProvider>
  </QueryClientProvider>
);

export default App;
```
1. **Provider Nesting Hierarchy:**
   * `QueryClientProvider`: Supplies cache management and async server queries across all children.
   * `ThemeProvider`: Applies `.dark` or `.light` classes to the document `<html>` tag based on system preference or user toggle.
   * `AuthProvider`: Exposes the current user session and authentication primitives to every route.
   * `TooltipProvider`: Supplies shared state for Radix UI hover tooltips.
   * Dual Toasters (`Toaster` and `Sonner`): Configured together to support standard notification banners and rich animated micro-toasts.
2. **Route Table Split:**
   * **Public Routes:** `/` (landing page) and `/auth` (login/register).
   * **Protected Routes:** Every assessment and profile route (`/quiz`, `/results/:assessmentId`, `/history`, `/profile`, `/favorites`, `/face-capture`) is wrapped in `<ProtectedRoute>`, preventing unauthenticated access.
   * **Wildcard Route:** `<Route path="*" element={<NotFound />} />` captures all invalid URLs and displays a 404 screen.

* **Concepts a beginner might not know:**
  * **Provider Pattern:** A React design pattern using React Context where an outer component makes data or functionality accessible to all descendant components regardless of how deeply nested they are, avoiding "prop drilling".
  * **Dynamic Route Parameters (`:assessmentId`):** URL variables denoted by a colon that match dynamic values in the browser address bar (e.g., `/results/abc-123`), accessible inside page components via `useParams()`.
  * **Client-Side Routing:** Intercepting browser URL changes with JavaScript so new pages render instantly without contacting an HTTP server for new HTML documents.

* **Why this approach, not an alternative:**
  * *Dual Toaster Integration (`Toaster` + `Sonner`):* While functional, maintaining both shadcn's original Radix-based toast (`Toaster`) and Emil Kowalski's `Sonner` is slightly redundant. In a clean production refactor, standardizing on Sonner alone would eliminate unused bundle weight.
  * *Route Wrapping vs. Layout Routes:* The codebase wraps individual routes with `<ProtectedRoute><Component /></ProtectedRoute>`. An alternative pattern is React Router's nested Layout Route (`<Route element={<ProtectedRouteLayout />}><Route ... /><Route ... /></Route>`). While wrapping individual routes is more verbose, it makes access rules explicitly obvious when reading the route table.

* **Connects to:**
  * **Imports from:** UI toasters, [AuthContext.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/contexts/AuthContext.jsx), [ThemeProvider.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/ThemeProvider.jsx), [ProtectedRoute.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/ProtectedRoute.jsx), and all 9 page components in `src/pages/`.
  * **Imported by:** [src/main.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/main.jsx).

---

### `src/lib/utils.js`

* **What it does:** Exports the universal class-name helper `cn()`. It combines conditional class application with intelligent Tailwind CSS conflict resolution.

* **Full walkthrough:**
```javascript
import { clsx, type ClassValue } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```
1. `clsx(inputs)` parses arrays, objects, booleans, and strings to strip out falsy values (e.g., `condition && 'text-red-500'`).
2. `twMerge(...)` inspects the resulting list of class strings, understands Tailwind CSS utility specificity, and automatically drops conflicting classes (e.g., `twMerge("px-2 py-1", "p-4")` produces `"p-4"` without manual string slicing).

* **Concepts a beginner might not know:**
  * **Rest Parameters (`...inputs`):** JavaScript syntax allowing a function to accept an indefinite number of arguments as an array.
  * **Tailwind Class Collisions:** In vanilla CSS, if you write `<div class="px-2 px-4">`, the rule that appears later in the compiled CSS stylesheet wins, not the one written last in your HTML attribute. `tailwind-merge` solves this by resolving conflicts deterministically at runtime.

* **Why this approach, not an alternative:**
  * *`clsx` + `twMerge` vs. template strings (`${...}`):* Template strings cannot resolve Tailwind class overrides passed down via `className` props. `cn()` is the established industry standard across modern React and shadcn/ui applications.

* **Connects to:**
  * **Imported by:** Virtually every visual component in the project: [BrandLogo.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/BrandLogo.jsx), [FavoriteButton.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/FavoriteButton.jsx), [ProductFeedback.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/ProductFeedback.jsx), [FaceCapture.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/FaceCapture.jsx), quiz step cards, and shadcn UI primitives.

---

### `src/types/skincare.js`

* **What it does:** Declares the domain TypeScript interfaces, data models, and static option matrices for the entire skincare recommendation system.

* **Full walkthrough:**
```javascript
export interface SkinAssessment {
  ageRange: string;
  gender: string;
  skinType: string;
  skinConcerns: string[];
  climate: string;
  allergies: string[];
  skinGoals: string;
  budgetRange: string;
}

export interface Product {
  name: string;
  brand: string;
  description: string;
  keyIngredients: string[];
  skinTypeSuitability: string[];
  priceRange: string;
  productUrl?: string;
  imageUrl?: string;
  whySuitable: string;
  usageInstructions?: string;
  safetyWarnings?: string;
}

export interface Recommendation {
  id: string;
  assessmentId: string;
  products: Product[];
  aiSummary: string;
  createdAt: string;
}

export const SKIN_TYPES = [
  { value: 'oily', label: 'Oily', description: 'Shiny, enlarged pores, prone to acne' },
  { value: 'dry', label: 'Dry', description: 'Tight, flaky, rough texture' },
  { value: 'combination', label: 'Combination', description: 'Oily T-zone, dry cheeks' },
  { value: 'sensitive', label: 'Sensitive', description: 'Easily irritated, reactive' },
  { value: 'normal', label: 'Normal', description: 'Balanced, few imperfections' },
] as const;

export const SKIN_CONCERNS = [
  { value: 'acne', label: 'Acne & Breakouts' },
  { value: 'pigmentation', label: 'Pigmentation & Dark Spots' },
  { value: 'dryness', label: 'Dryness & Dehydration' },
  { value: 'aging', label: 'Fine Lines & Wrinkles' },
  { value: 'redness', label: 'Redness & Rosacea' },
  { value: 'dullness', label: 'Dullness & Uneven Tone' },
  { value: 'pores', label: 'Large Pores' },
  { value: 'oiliness', label: 'Excess Oil' },
  { value: 'texture', label: 'Rough Texture' },
  { value: 'darkCircles', label: 'Dark Circles' },
] as const;

export const AGE_RANGES = [
  { value: 'under18', label: 'Under 18' },
  { value: '18-24', label: '18-24' },
  { value: '25-34', label: '25-34' },
  { value: '35-44', label: '35-44' },
  { value: '45-54', label: '45-54' },
  { value: '55+', label: '55+' },
] as const;

export const CLIMATES = [
  { value: 'tropical', label: 'Tropical (Hot & Humid)' },
  { value: 'dry', label: 'Dry / Arid' },
  { value: 'temperate', label: 'Temperate' },
  { value: 'cold', label: 'Cold / Winter' },
  { value: 'varied', label: 'Varied / Seasonal' },
] as const;

export const BUDGET_RANGES = [
  { value: 'budget', label: 'Budget-Friendly (Under ₹500)' },
  { value: 'mid', label: 'Mid-Range (₹500–₹1500)' },
  { value: 'premium', label: 'Premium (₹1500–₹3000)' },
  { value: 'luxury', label: 'Luxury (₹3000+)' },
] as const;

export const COMMON_ALLERGIES = [
  { value: 'fragrance', label: 'Fragrance' },
  { value: 'parabens', label: 'Parabens' },
  { value: 'sulfates', label: 'Sulfates' },
  { value: 'alcohol', label: 'Alcohol' },
  { value: 'retinol', label: 'Retinol' },
  { value: 'vitaminC', label: 'Vitamin C' },
  { value: 'salicylicAcid', label: 'Salicylic Acid' },
  { value: 'niacinamide', label: 'Niacinamide' },
] as const;
```
1. **Domain Interfaces:** `SkinAssessment` captures user inputs across demographics, dermatology, and budget. `Product` specifies properties returned by the AI engine. `Recommendation` maps the relationship between an assessment and recommended products.
2. **Options Matrices (`as const`):** Defines constant lookup lists for UI radios and checkboxes, covering skin types, concerns, climates, Indian Rupee (`₹`) budget tiers, and common skincare allergen sensitizers.

* **Concepts a beginner might not know:**
  * **TypeScript Interface:** A syntactical contract that defines the exact structure and types of properties an object must adhere to at compile time.
  * **Const Assertions (`as const`):** A TypeScript modifier that locks an object or array into a read-only literal type, ensuring properties cannot be reassigned and allowing TypeScript to infer exact string literal union types instead of broad `string` types.
  * **Optional Properties (`?`):** Denotes that a field may be omitted or undefined (e.g., `productUrl?: string`).

* **Why this approach, not an alternative:**
  * *Centralized Domain Schema:* Putting domain types and constants in a single agnostic file ensures frontend forms, custom hooks, and mock utilities share a single source of truth.
  * *Critique / Weakness:* Notice that field names in `SkinAssessment` use camelCase (`ageRange`, `skinType`, `budgetRange`), whereas PostgreSQL columns in Supabase use snake_case (`age_range`, `skin_type`, `budget_range`). This causes manual field mapping in `Quiz.jsx` and `Results.jsx`. A stricter schema approach using a unified Zod schema with automated naming transforms would prevent subtle mapping bugs.

* **Connects to:**
  * **Imported by:** [useAssessment.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/hooks/useAssessment.js), all quiz components in `src/components/quiz/`, [Results.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Results.jsx), [Favorites.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Favorites.jsx), [History.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/History.jsx), and [FavoriteButton.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/FavoriteButton.jsx).

---

### `src/integrations/supabase/client.js`

* **What it does:** Instantiates and exports the singleton Supabase client configured with the application's environment keys, database types, and browser session storage persistence.

* **Full walkthrough:**
```javascript
// This file is automatically generated. Do not edit it directly.
import { createClient } from '@supabase/supabase-js';
import type { Database } from './types';

const SUPABASE_URL = import.meta.env.VITE_SUPABASE_URL;
const SUPABASE_PUBLISHABLE_KEY = import.meta.env.VITE_SUPABASE_PUBLISHABLE_KEY;

// Import the supabase client like this:
// import { supabase } from "@/integrations/supabase/client";

export const supabase = createClient<Database>(SUPABASE_URL, SUPABASE_PUBLISHABLE_KEY, {
  auth: {
    storage: localStorage,
    persistSession: true,
    autoRefreshToken: true,
  }
});
```
1. **Environment Variables:** Reads `VITE_SUPABASE_URL` and `VITE_SUPABASE_PUBLISHABLE_KEY` from Vite's `import.meta.env` context.
2. **Generic Database Typing (`<Database>`):** Passes the auto-generated database interface from `types.js` into `createClient<Database>()`. This provides end-to-end type safety when querying tables (e.g., `supabase.from('skin_assessments')` knows every column and return type).
3. **Session Configuration:**
   * `storage: localStorage`: Persists auth JWT tokens in browser storage across tab refreshes.
   * `persistSession: true`: Automatically restores sessions upon page reload.
   * `autoRefreshToken: true`: Automatically requests a fresh JWT token before the current token expires.

* **Concepts a beginner might not know:**
  * **`import.meta.env`:** Vite's mechanism for exposing environment variables to client-side code. Only variables prefixed with `VITE_` are bundled into frontend code to prevent accidentally leaking private backend keys.
  * **JWT (JSON Web Token):** A cryptographically signed digital token passed in HTTP authorization headers proving the caller's identity without requiring a database session lookup on every request.
  * **Singleton Pattern:** A software design pattern that restricts the instantiation of a class to one single global instance across the application lifecycle.

* **Why this approach, not an alternative:**
  * *Direct Client Instantiation:* Provides zero-overhead access to database queries directly from React components and hooks protected by Row Level Security (RLS).
  * *Critique / Defense Point:* Notice that if `VITE_SUPABASE_URL` or `VITE_SUPABASE_PUBLISHABLE_KEY` is missing (e.g., during initial setup or misconfigured deployment), `createClient` receives `undefined` and may fail silently or throw runtime errors during boot. A runtime sanity check throwing a descriptive error message would be significantly safer.

* **Connects to:**
  * **Imports from:** `@supabase/supabase-js` and [types.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/integrations/supabase/types.js).
  * **Imported by:** [AuthContext.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/contexts/AuthContext.jsx), [Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx), [Results.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Results.jsx), [History.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/History.jsx), [Favorites.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Favorites.jsx), [Profile.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Profile.jsx), [FavoriteButton.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/FavoriteButton.jsx), and [ProductFeedback.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/ProductFeedback.jsx).

---

### `src/contexts/AuthContext.jsx`

* **What it does:** Serves as the central security and authentication state manager for the frontend. It wraps Supabase's GoTrue authentication client, provides real-time auth state synchronization, exposes `signUp`, `signIn`, and `signOut` methods, and distributes the active user session across the React tree.

* **Full walkthrough:**
```javascript
import { createContext, useContext, useEffect, useState, ReactNode } from 'react';
import { User, Session } from '@supabase/supabase-js';
import { supabase } from '@/integrations/supabase/client';

interface AuthContextType {
  user: User | null;
  session: Session | null;
  loading: boolean;
  signUp: (email: string, password: string, fullName?: string) => Promise<{ error: Error | null }>;
  signIn: (email: string, password: string) => Promise<{ error: Error | null }>;
  signOut: () => Promise<void>;
}

const AuthContext = createContext<AuthContextType | undefined>(undefined);
```
The context defines an interface `AuthContextType` providing the `user`, `session`, a `loading` flag, and asynchronous auth actions. The context is initialized with `undefined` to force consumers to execute within an `<AuthProvider>` wrapper.

```javascript
export function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [session, setSession] = useState<Session | null>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Set up auth state listener FIRST
    const { data: { subscription } } = supabase.auth.onAuthStateChange(
      (event, session) => {
        setSession(session);
        setUser(session?.user ?? null);
        setLoading(false);
      }
    );

    // THEN check for existing session
    supabase.auth.getSession().then(({ data: { session } }) => {
      setSession(session);
      setUser(session?.user ?? null);
      setLoading(false);
    });

    return () => subscription.unsubscribe();
  }, []);
```
1. **Critical Listener Architecture:** Notice the comment: `Set up auth state listener FIRST` then `check for existing session`. This deliberate ordering prevents race conditions where an active session is loaded from `localStorage` before the subscription listener is active to receive subsequent token refreshes or login events.
2. **Unsubscription Cleanup:** `return () => subscription.unsubscribe();` ensures that when the provider unmounts, the WebSocket/broadcast event listener is destroyed to prevent memory leaks.

```javascript
  const signUp = async (email: string, password: string, fullName?: string) => {
    const redirectUrl = `${window.location.origin}/`;

    const { error } = await supabase.auth.signUp({
      email,
      password,
      options: {
        emailRedirectTo: redirectUrl,
        data: {
          full_name: fullName || '',
        },
      },
    });

    return { error: error as Error | null };
  };

  const signIn = async (email: string, password: string) => {
    const { error } = await supabase.auth.signInWithPassword({
      email,
      password,
    });

    return { error: error as Error | null };
  };
```
3. `signUp()` passes `emailRedirectTo: ${window.location.origin}/` so that confirmation email links redirect back to the app's root URL. User metadata (`full_name`) is embedded directly into Supabase auth's `raw_user_meta_data` JSON column.
4. `signIn()` invokes `signInWithPassword()`, which sends credentials to Supabase GoTrue over HTTPS and returns the signed JWT session.

```javascript
  const signOut = async () => {
    try {
      await supabase.auth.signOut();
    } catch (error) {
      console.error('Error signing out:', error);
    } finally {
      // Force clear any supabase auth tokens from localStorage
      for (const key of Object.keys(localStorage)) {
        if (key.includes('supabase') || key.includes('-auth-token')) {
          localStorage.removeItem(key);
        }
      }
      setUser(null);
      setSession(null);
    }
  };

  return (
    <AuthContext.Provider value={{ user, session, loading, signUp, signIn, signOut }}>
      {children}
    </AuthContext.Provider>
  );
}

export function useAuth() {
  const context = useContext(AuthContext);
  if (context === undefined) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
}
```
5. **Defensive Logout (`signOut`):** Even if the network call `supabase.auth.signOut()` fails (e.g., user is offline), the `finally` block loops over `localStorage` keys and nukes all tokens matching `supabase` or `-auth-token`, guaranteeing the user is logged out locally.
6. **Hook Guard (`useAuth`):** Enforces defensive consumption by throwing an explicit runtime error if `useAuth()` is called outside an `<AuthProvider>`.

* **Concepts a beginner might not know:**
  * **React Context API:** A built-in React mechanism for sharing global values between components without passing props down through intermediate parent components manually.
  * **Race Condition:** A flaw where the timing or sequence of events (such as two asynchronous network or disk promises completing in an unexpected order) affects the correctness of program execution.
  * **Memory Leak:** When allocated memory (or an open network listener) is not released when no longer needed, causing performance degradation over time.
  * **GoTrue:** Supabase's open-source authentication server written in Go that manages user registrations, email confirmations, password hashes (bcrypt), and OAuth providers.

* **Why this approach, not an alternative:**
  * *Manual localStorage Token Purge:* In `signOut()`, looping through `localStorage` is an aggressive defensive safeguard. Standard Supabase client libraries clear their own tokens automatically; however, if network requests timeout or fail, the native SDK can leave stale tokens behind. The manual loop guarantees logout completion.
  * *Context vs. Redux / Zustand:* For auth state (which changes infrequently and is consumed app-wide), standard React Context has zero third-party overhead compared to heavier state stores like Redux.

* **Connects to:**
  * **Imports from:** `@supabase/supabase-js` and [client.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/integrations/supabase/client.js).
  * **Imported by:** [App.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/App.jsx) (wraps the route tree), [ProtectedRoute.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/ProtectedRoute.jsx), [Index.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Index.jsx), [Auth.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Auth.jsx), [Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx), [Results.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Results.jsx), [History.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/History.jsx), [Profile.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Profile.jsx), and [Favorites.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Favorites.jsx).

---

### `src/components/ProtectedRoute.jsx`

* **What it does:** An authorization higher-order component that wraps sensitive application routes. It prevents unauthenticated visitors from viewing private screens while preventing "flash redirects" while the auth session initializes.

* **Full walkthrough:**
```javascript
import { Navigate } from 'react-router-dom';
import { useAuth } from '@/contexts/AuthContext';

interface ProtectedRouteProps {
  children: React.ReactNode;
}

/**
 * Wraps any route that requires authentication.
 * While the auth state is loading, renders nothing (avoids a flash redirect).
 * Once resolved, redirects unauthenticated visitors to /auth.
 */
export default function ProtectedRoute({ children }: ProtectedRouteProps) {
  const { user, loading } = useAuth();

  if (loading) {
    return null; // or a spinner – keeps the screen blank until auth resolves
  }

  if (!user) {
    return <Navigate to="/auth" replace />;
  }

  return <>{children}</>;
}
```
1. Reads `user` and `loading` from `useAuth()`.
2. **Flash Prevention:** If `loading === true`, it immediately returns `null`. This prevents the application from mistakenly redirecting a logged-in user to `/auth` during the ~50 milliseconds it takes to read their session from `localStorage`.
3. **Redirect:** If `user` is null after loading finishes, it renders `<Navigate to="/auth" replace />`. The `replace` flag replaces the current history entry so pressing the browser's Back button does not loop the user into an infinite redirect loop.
4. **Pass-through:** If authenticated, renders the child component tree cleanly.

* **Concepts a beginner might not know:**
  * **Flash Redirect (FOUC):** A brief visual glitch where a user is incorrectly bounced to a login page because the app checked for user credentials before local storage finished parsing the auth session.
  * **`replace` vs `push` Navigation:** A standard navigation (`push`) appends a new URL onto the browser's history stack. A `replace` navigation overwrites the current URL slot, preventing broken Back button loops.
  * **Higher-Order Component / Wrapper:** A component that accepts other components as children to conditionally inject behaviors, permissions, or styling.

* **Why this approach, not an alternative:**
  * *`return null` vs. Loading Spinner:* The component returns `null` during loading. While this avoids showing a redirect, in slow network environments or cold boot, rendering an explicit skeleton or animated spinner would provide better visual feedback to the user than a blank white/cream screen.

* **Connects to:**
  * **Imports from:** `react-router-dom` and [AuthContext.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/contexts/AuthContext.jsx).
  * **Imported by:** [App.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/App.jsx) (wraps `/quiz`, `/results/:assessmentId`, `/history`, `/profile`, `/favorites`, and `/face-capture`).

---

### `src/pages/Auth.jsx`

* **What it does:** Renders the authentication interface supporting both login and new account registration. It implements client-side form validation via Zod schemas and manages user redirects.

* **Full walkthrough:**
```javascript
import { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '@/contexts/AuthContext';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Card, CardContent, CardDescription, CardFooter, CardHeader, CardTitle } from '@/components/ui/card';
import { useToast } from '@/hooks/use-toast';
import { Mail, Lock, User } from 'lucide-react';
import { BrandLogo } from '@/components/BrandLogo';
import { z } from 'zod';

const authSchema = z.object({
  email: z.string().email('Please enter a valid email address'),
  password: z.string().min(6, 'Password must be at least 6 characters'),
  fullName: z.string().optional(),
});
```
1. **Validation Schema (`authSchema`):** Declares strict criteria using Zod: valid email format and minimum 6-character password constraint.

```javascript
export default function Auth() {
  const [isSignUp, setIsSignUp] = useState(false);
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [fullName, setFullName] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [errors, setErrors] = useState<{ email?: string; password?: string }>({});
  
  const { signIn, signUp, user, loading } = useAuth();
  const navigate = useNavigate();
  const { toast } = useToast();

  useEffect(() => {
    if (!loading && user) {
      navigate('/');
    }
  }, [user, loading, navigate]);

  const validateForm = () => {
    try {
      authSchema.parse({ email, password, fullName });
      setErrors({});
      return true;
    } catch (err) {
      if (err instanceof z.ZodError) {
        const fieldErrors: { email?: string; password?: string } = {};
        err.errors.forEach((error) => {
          if (error.path[0] === 'email') fieldErrors.email = error.message;
          if (error.path[0] === 'password') fieldErrors.password = error.message;
        });
        setErrors(fieldErrors);
      }
      return false;
    }
  };
```
2. **Reverse Redirect:** If an already authenticated user navigates to `/auth`, the `useEffect` hook immediately navigates them to the home page (`/`).
3. **Zod Validation Handler:** `validateForm()` parses the state values through `authSchema.parse()`. If validation fails, it extracts errors by path (`error.path[0]`) and populates the `errors` state object to render red error messages under individual inputs.

```javascript
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    if (!validateForm()) return;
    setIsLoading(true);

    try {
      if (isSignUp) {
        const { error } = await signUp(email, password, fullName);
        if (error) {
          if (error.message.includes('already registered')) {
            toast({
              title: 'Account exists',
              description: 'This email is already registered. Please sign in instead.',
              variant: 'destructive',
            });
          } else {
            toast({
              title: 'Sign up failed',
              description: error.message,
              variant: 'destructive',
            });
          }
        } else {
          toast({
            title: 'Welcome to SkinWise!',
            description: 'Your account has been created successfully.',
          });
          navigate('/');
        }
      } else {
        const { error } = await signIn(email, password);
        if (error) {
          toast({
            title: 'Sign in failed',
            description: 'Invalid email or password. Please try again.',
            variant: 'destructive',
          });
        } else {
          navigate('/');
        }
      }
    } finally {
      setIsLoading(false);
    }
  };
```
4. **Submission & Error Dissection:** Submits either to `signUp()` or `signIn()`. It specifically inspects `error.message` for `'already registered'` and guides the user to sign in rather than showing a generic cryptic error.

* **Concepts a beginner might not know:**
  * **Zod:** A TypeScript-first schema declaration and data validation library that checks object structures at runtime and generates static types automatically.
  * **`e.preventDefault()`:** Prevents the browser's default behavior when submitting an HTML `<form>`, which would otherwise trigger an HTTP POST request and reload the entire webpage.
  * **Controlled Components:** Form inputs whose values are bound directly to React state (`value={email}`) and updated via an `onChange` callback, making React the single source of truth for the input's current value.

* **Why this approach, not an alternative:**
  * *Toggleable Single Auth Screen vs. Two Pages (`/login` and `/signup`):* Combining both modes into a single page with `isSignUp` state creates a smoother user experience with zero navigation latency.
  * *Critique / Security Improvement:* The password input enforces `min(6)` on the client. For a production healthcare/skincare application, adding regex rules for capital letters, numbers, and special characters would defend against weak credentials.

* **Connects to:**
  * **Imports from:** [AuthContext.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/contexts/AuthContext.jsx), [use-toast.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/hooks/use-toast.js), [BrandLogo.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/BrandLogo.jsx), and shadcn Card/Input/Button UI components.
  * **Referenced by:** [App.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/App.jsx) and [ProtectedRoute.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/ProtectedRoute.jsx).

---

### `src/hooks/useFaceDetection.js`

* **What it does:** The algorithmic engine of the application's Computer Vision quality gating pipeline. It executes four sequential image quality checks directly inside the client browser: minimum resolution verification, Rec. 709 luminance exposure gating, discrete Laplacian convolution blur detection, and TensorFlow.js BlazeFace facial landmark detection and oval bounding containment.

* **Full walkthrough:**
```javascript
import { useRef, useCallback } from 'react';
import * as tf from '@tensorflow/tfjs';
import * as blazeface from '@tensorflow-models/blazeface';

let modelPromise: Promise<blazeface.BlazeFaceModel> | null = null;

function getModel() {
  if (!modelPromise) {
    modelPromise = tf.ready().then(() => blazeface.load());
  }
  return modelPromise;
}

export interface FaceCheckResult {
  passed: boolean;
  message: string;
  tip: string;
}

interface OvalBounds {
  cx: number;
  cy: number;
  rx: number;
  ry: number;
}
```
1. **Module-Scoped Singleton Promise (`modelPromise`):** BlazeFace model weights are relatively heavy (~1–2 MB binary graph). By storing the initialization promise outside the React hook in module scope, the deep learning weights are fetched and loaded into the browser's WebGL context exactly once across the user's session, avoiding repeated multi-megabyte downloads.

```javascript
/**
 * Compute average luminance (0-255) from RGBA image data.
 */
function computeLuminance(data: Uint8ClampedArray): number {
  let sum = 0;
  const pixelCount = data.length / 4;
  for (let i = 0; i < data.length; i += 4) {
    // Rec. 709 luminance
    sum += 0.2126 * data[i] + 0.7152 * data[i + 1] + 0.0722 * data[i + 2];
  }
  return sum / pixelCount;
}
```
2. **ITU-R BT.709 Luminance Standard:** The function iterates over the raw 1D pixel byte buffer (`Uint8ClampedArray`) where every pixel comprises 4 consecutive bytes `[R, G, B, A]`. Instead of a naive unweighted average `(R + G + B) / 3`, it applies standard photometric coefficients:
   $$\text{Luminance} = 0.2126 \cdot R + 0.7152 \cdot G + 0.0722 \cdot B$$
   Green receives the heaviest weight ($71.52\%$) because human eye cone cells are most sensitive to green wavelengths. The resulting score ($0-255$) is checked against thresholds: $< 50$ indicates underexposure ("Too dark"), while $> 220$ indicates blowout overexposure ("Too bright").

```javascript
/**
 * Simple blur detection via variance of Laplacian on grayscale image.
 * For production, replace with OpenCV.js cv.Laplacian for better accuracy.
 */
function computeLaplacianVariance(
  data: Uint8ClampedArray,
  width: number,
  height: number
): number {
  // Convert to grayscale
  const gray = new Float32Array(width * height);
  for (let i = 0; i < width * height; i++) {
    const idx = i * 4;
    gray[i] = 0.299 * data[idx] + 0.587 * data[idx + 1] + 0.114 * data[idx + 2];
  }

  // Apply 3x3 Laplacian kernel [0,1,0; 1,-4,1; 0,1,0]
  let sum = 0;
  let sumSq = 0;
  let count = 0;
  for (let y = 1; y < height - 1; y++) {
    for (let x = 1; x < width - 1; x++) {
      const lap =
        gray[(y - 1) * width + x] +
        gray[(y + 1) * width + x] +
        gray[y * width + (x - 1)] +
        gray[y * width + (x + 1)] -
        4 * gray[y * width + x];
      sum += lap;
      sumSq += lap * lap;
      count++;
    }
  }
  const mean = sum / count;
  return sumSq / count - mean * mean;
}
```
3. **Variance of Laplacian Blur Metric:**
   * Step A: Converts RGB pixels into a continuous grayscale floating-point buffer (`Float32Array`).
   * Step B: Convolves a standard 2D discrete Laplacian edge-detection kernel across every non-boundary pixel:
     $$\mathbf{K} = \begin{bmatrix} 0 & 1 & 0 \\ 1 & -4 & 1 \\ 0 & 1 & 0 \end{bmatrix}$$
   * Step C: Calculates the statistical variance ($\sigma^2 = \frac{\sum x^2}{N} - \mu^2$). High variance indicates crisp, sharp transitions along skin pores and facial edges. Low variance ($< 120$) indicates blurred, motion-degraded pixels.

```javascript
function isBboxInsideOval(
  topLeft: [number, number],
  bottomRight: [number, number],
  oval: OvalBounds,
  threshold = 0.7
): boolean {
  const faceW = bottomRight[0] - topLeft[0];
  const faceH = bottomRight[1] - topLeft[1];
  const faceCx = topLeft[0] + faceW / 2;
  const faceCy = topLeft[1] + faceH / 2;

  // Check center of face is inside oval
  const dx = (faceCx - oval.cx) / oval.rx;
  const dy = (faceCy - oval.cy) / oval.ry;
  if (dx * dx + dy * dy > 1) return false;

  // Check corners overlap ratio — simplified: face area vs oval area
  const faceArea = faceW * faceH;
  const ovalArea = Math.PI * oval.rx * oval.ry;
  return faceArea / ovalArea > 0.15 && faceArea / ovalArea < 0.95;
}
```
4. **Geometric Oval Ellipse Containment:** Checks if the face bounding box center `(faceCx, faceCy)` lies inside the target guide ellipse using the normalized ellipse inequality:
   $$\frac{\Delta x^2}{r_x^2} + \frac{\Delta y^2}{r_y^2} \le 1$$
   It also ensures the face fills between $15\%$ and $95\%$ of the oval area, preventing faces that are too far away or pressed right against the camera lens.

```javascript
export function useFaceDetection() {
  const modelRef = useRef<blazeface.BlazeFaceModel | null>(null);

  const runChecks = useCallback(
    async (
      canvas: HTMLCanvasElement,
      ovalBounds: OvalBounds
    ): Promise<FaceCheckResult> => {
      const ctx = canvas.getContext('2d')!;
      const { width, height } = canvas;

      // 1. Resolution check
      if (width < 800) {
        return {
          passed: false,
          message: 'Low resolution',
          tip: 'Use a higher resolution camera or move closer.',
        };
      }

      const imageData = ctx.getImageData(0, 0, width, height);
      const { data } = imageData;

      // 2. Luminance check
      const lum = computeLuminance(data);
      if (lum < 50) return { passed: false, message: 'Too dark', tip: 'Too dark — move to brighter, indirect light.' };
      if (lum > 220) return { passed: false, message: 'Too bright/overexposed', tip: 'Too bright/overexposed — avoid direct sunlight or bright lamps.' };

      // 3. Blur check
      const blurScore = computeLaplacianVariance(data, width, height);
      if (blurScore < 120) return { passed: false, message: 'Blurry', tip: 'Blurry — hold the phone steady or tap to focus.' };

      // 4. Face detection
      try {
        if (!modelRef.current) {
          modelRef.current = await getModel();
        }
        const predictions = await modelRef.current.estimateFaces(canvas, false);

        if (predictions.length === 0) return { passed: false, message: 'Face not detected', tip: 'Area not detected — ensure your face is centered inside the oval.' };
        if (predictions.length > 1) return { passed: false, message: 'Multiple faces detected', tip: 'Multiple faces detected — only one person in the frame please.' };

        const face = predictions[0];
        if ((face.probability as number) < 0.75) return { passed: false, message: 'Face partly occluded', tip: 'Face partly occluded — remove hands, hair, or accessories covering your face.' };

        const topLeft = face.topLeft as [number, number];
        const bottomRight = face.bottomRight as [number, number];

        if (!isBboxInsideOval(topLeft, bottomRight, ovalBounds)) {
          return { passed: false, message: 'Face not centered', tip: 'Area not detected — ensure your face is centered inside the oval.' };
        }
      } catch (err) {
        console.error('Face detection error:', err);
        return { passed: false, message: 'Detection error', tip: 'Could not analyze the image. Please try again.' };
      }

      return { passed: true, message: 'Checks passed', tip: 'Checks passed — sending photo for analysis.' };
    },
    []
  );

  return { runChecks };
}
```
5. **Quality Gate Execution Pipeline:**
   * **Step 1 — Resolution Gating:** Drops images with horizontal pixel count $< 800$ to ensure generative AI models can discern fine skin texture.
   * **Step 2 — Luminance Gating:** Flags underexposure ($< 50$) and blowout glare ($> 220$).
   * **Step 3 — Blur Gating:** Flags blurred images ($< 120$).
   * **Step 4 — BlazeFace Deep Learning Inference:** Detects faces with `estimateFaces()`. Rejects images with 0 faces, multiple faces ($> 1$), low prediction probability ($< 0.75$, indicative of occlusion like face masks), or faces outside the oval boundaries.

* **Concepts a beginner might not know:**
  * **Convolution Kernel:** A small matrix (like the 3x3 Laplacian) slid over every pixel of an image to compute mathematical derivatives for edge detection, blurring, or sharpening.
  * **Typed Arrays (`Uint8ClampedArray`, `Float32Array`):** High-performance JavaScript array objects designed specifically to handle raw binary numeric memory buffers without normal JavaScript array overhead.
  * **BlazeFace:** A lightweight deep neural network created by Google Research specifically optimized for sub-millisecond mobile GPU/WebGL inference of facial bounding boxes and 6 key landmark coordinates.
  * **Confidence Score (`probability`):** A value between $0.0$ and $1.0$ indicating how certain the neural network is that a detected pattern is genuinely a human face.

* **Why this approach, not an alternative:**
  * *Client-Side CV Gating vs. Server-Side Python OpenCV:* Validating images on the client using WebGL saves enormous backend computing costs and cloud bandwidth. If a user uploads a blurry or dark photo, they get instant millisecond feedback without wasting a costly Gemini API LLM invocation.
  * *Custom JS Laplacian vs. OpenCV.js WebAssembly:* The comment in line 42 candidly notes that OpenCV.js provides higher mathematical fidelity. However, including OpenCV.js would inflate the client bundle by 8–10 MB. Writing the discrete 3x3 convolution loop in pure TypeScript achieves acceptable blur gating with zero external library overhead!

* **Connects to:**
  * **Imports from:** `@tensorflow/tfjs` and `@tensorflow-models/blazeface`.
  * **Imported by:** [FaceCapture.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/FaceCapture.jsx).

---

### `src/components/FaceCapture.jsx`

* **What it does:** The user interface and hardware controller for the computer vision pipeline. It interfaces with the browser's MediaDevices API to stream live webcam video, enumerates camera hardware, renders a dynamic SVG alignment oval overlay, coordinates capture triggers, and collects GDPR/HIPAA privacy consent.

* **Full walkthrough:**
```javascript
import { useState, useRef, useCallback, useEffect } from 'react';
import { Button } from '@/components/ui/button';
import { Card, CardContent } from '@/components/ui/card';
import { Checkbox } from '@/components/ui/checkbox';
import { Label } from '@/components/ui/label';
import { Badge } from '@/components/ui/badge';
import { Camera, Upload, X, RotateCcw, Loader2, CheckCircle2, AlertTriangle } from 'lucide-react';
import { useFaceDetection, type FaceCheckResult } from '@/hooks/useFaceDetection';

interface FaceCaptureProps {
  onValidCapture: (base64Image: string) => void;
  onCancel?: () => void;
}
```
1. **Component State and Refs:** Tracks the live stream in `streamRef`, attaches the hardware stream to an HTML `<video>` element via `videoRef`, captures frames onto an offscreen `<canvas>` via `canvasRef`, and maintains states for camera initialization, active device switching, CV check results, captured base64 data, and privacy consent.

```javascript
  const startCamera = useCallback(async (deviceId?: string) => {
    setError(null);
    setResult(null);
    setCapturedImage(null);
    setCameraInitializing(true);
    setCameraActive(true);

    try {
      streamRef.current?.getTracks().forEach((track) => track.stop());

      const constraints: MediaStreamConstraints = deviceId
        ? {
            video: {
              deviceId: { exact: deviceId },
              width: { ideal: 1280 },
              height: { ideal: 960 },
            },
          }
        : {
            video: { facingMode: 'user', width: { ideal: 1280 }, height: { ideal: 960 } },
          };

      let stream: MediaStream;
      try {
        stream = await navigator.mediaDevices.getUserMedia(constraints);
      } catch {
        // Fallback for devices/browsers that can't satisfy preferred constraints
        stream = await navigator.mediaDevices.getUserMedia({ video: true });
      }

      streamRef.current = stream;
      setCameraStream(stream);

      const cameras = await refreshVideoDevices();
      const currentDeviceId = stream.getVideoTracks()[0]?.getSettings().deviceId;
      if (currentDeviceId) {
        const foundIndex = cameras.findIndex((cam) => cam.deviceId === currentDeviceId);
        if (foundIndex >= 0) setActiveDeviceIndex(foundIndex);
      }
    } catch (err) {
      const message = err instanceof Error ? `${err.name}: ${err.message}` : 'Unknown camera error';
      setCameraActive(false);
      setCameraStream(null);
      setCameraInitializing(false);
      setError(`Could not access camera. ${message}`);
    }
  }, [refreshVideoDevices]);
```
2. **Hardware Stream Management:**
   * Stops existing tracks before requesting new ones (`streamRef.current?.getTracks().forEach(track => track.stop())`) to free device camera hardware.
   * Requests an ideal resolution of 1280x960 with front-facing camera preference (`facingMode: 'user'`).
   * **Graceful Fallback:** If the user's camera hardware cannot satisfy exact 1280x960 constraints, it catches the error and falls back to generic `{ video: true }`.
   * Enumerates device inputs via `navigator.mediaDevices.enumerateDevices()` to support multi-camera switching on smartphones.

```javascript
  // Attach stream to video element when both are ready
  useEffect(() => {
    const video = videoRef.current;
    if (!cameraActive || !cameraStream || !video) return;

    let cancelled = false;
    const markReady = () => {
      if (cancelled) return;
      setCameraInitializing(false);
      setError(null);
    };

    const startPlayback = async () => {
      try {
        await video.play();
      } catch {
        // Ignore autoplay interruptions; user can retry capture
      }
    };

    video.srcObject = cameraStream;
    video.onplaying = markReady;

    if (video.readyState >= HTMLMediaElement.HAVE_METADATA) {
      void startPlayback();
    } else {
      video.onloadedmetadata = () => {
        void startPlayback();
      };
    }

    const watchdog = window.setTimeout(async () => {
      if (cancelled || video.readyState >= HTMLMediaElement.HAVE_CURRENT_DATA) return;
      try {
        video.srcObject = null;
        video.srcObject = cameraStream;
        await video.play();
      } catch {}

      if (!cancelled && video.readyState < HTMLMediaElement.HAVE_CURRENT_DATA) {
        setCameraInitializing(false);
        setError('Camera opened but no live feed was received. Try switching camera or close other camera apps, then retry.');
      }
    }, 3000);

    return () => {
      cancelled = true;
      window.clearTimeout(watchdog);
      video.onloadedmetadata = null;
      video.onplaying = null;
    };
  }, [cameraActive, cameraStream]);
```
3. **Black Screen Watchdog Timer:** Some mobile web browsers experience race conditions where `getUserMedia` resolves but the `<video>` element hangs on a black frame. The component attaches a 3-second watchdog timer: if `video.readyState < HAVE_CURRENT_DATA`, it retries binding once and presents a friendly error rather than leaving the user stuck on an unresponsive loader.

```javascript
  const capture = useCallback(async () => {
    if (!videoRef.current || !canvasRef.current) return;
    setChecking(true);
    setResult(null);

    const video = videoRef.current;
    const canvas = canvasRef.current;
    canvas.width = video.videoWidth;
    canvas.height = video.videoHeight;
    const ctx = canvas.getContext('2d')!;
    ctx.drawImage(video, 0, 0);

    const oval = getOvalBounds(canvas.width, canvas.height);
    const checkResult = await runChecks(canvas, oval);
    setResult(checkResult);

    if (checkResult.passed) {
      const base64 = canvas.toDataURL('image/jpeg', 0.92);
      setCapturedImage(base64);
      stopCamera();
    }
    setChecking(false);
  }, [runChecks, getOvalBounds, stopCamera]);
```
4. **Frame Capture & Quality Verification:**
   * Matches `<canvas>` dimensions to the physical camera stream dimensions (`video.videoWidth` and `video.videoHeight`).
   * Renders the frame onto canvas using `drawImage()`.
   * Evaluates the image against `runChecks(canvas, oval)`.
   * If quality criteria are met, encodes the frame into JPEG format at 92% quality (`canvas.toDataURL('image/jpeg', 0.92)`) and terminates the camera hardware stream to save battery.

```javascript
              {/* Oval overlay */}
              <svg
                className="absolute inset-0 w-full h-full pointer-events-none"
                viewBox="0 0 100 133"
                preserveAspectRatio="none"
                aria-hidden="true"
              >
                <defs>
                  <mask id="oval-mask">
                    <rect width="100" height="133" fill="white" />
                    <ellipse cx="50" cy="66.5" rx="32" ry="42" fill="black" />
                  </mask>
                </defs>
                <rect
                  width="100"
                  height="133"
                  fill="hsl(var(--foreground) / 0.45)"
                  mask="url(#oval-mask)"
                />
                <ellipse
                  cx="50"
                  cy="66.5"
                  rx="32"
                  ry="42"
                  fill="none"
                  stroke="hsl(var(--primary))"
                  strokeWidth="0.6"
                  strokeDasharray="3 2"
                />
              </svg>
```
5. **SVG Mask Cutout:** Utilizes an SVG `mask` containing a black ellipse over a white rectangle to render a dimmed, semi-transparent backdrop around an oval viewport, providing visual guidance for face positioning.

* **Concepts a beginner might not know:**
  * **MediaStream API:** The browser interface representing a stream of real-time media content (audio/video tracks) piped directly from physical webcams or microphones.
  * **`toDataURL`:** An HTMLCanvasElement method that converts the graphical contents of a canvas into a Base64-encoded Data URL string (e.g., `data:image/jpeg;base64,/9j/4AAQ...`).
  * **SVG Masking:** A vector graphics technique where the luminance or color of a mask shape controls the transparency of another graphic (black areas become completely transparent, creating a "see-through hole").

* **Why this approach, not an alternative:**
  * *Dual Input (Live Camera + File Upload):* Provides both real-time webcam streaming and file picker upload (`handleFileUpload`). This ensures users on laptops without working webcams or mobile devices with restrictive browser permissions can still upload an existing gallery photo.
  * *Mandatory Privacy Consent Checkbox:* Skincare face photos constitute sensitive biometric health data. Requiring explicit checkbox consent before passing `base64Image` to `onValidCapture` demonstrates ethical privacy engineering.

* **Connects to:**
  * **Imports from:** [useFaceDetection.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/hooks/useFaceDetection.js), shadcn UI components, and Lucide icons.
  * **Imported by:** [StepFacePhoto.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/quiz/StepFacePhoto.jsx) (Quiz Step 6) and [FaceCapturePage.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/FaceCapturePage.jsx).

---

### `src/pages/FaceCapturePage.jsx`

* **What it does:** A dedicated testing and diagnostic page for the face capture system. It mounts `<FaceCapture>` in isolation with a brand header and toast feedback.

* **Full walkthrough:**
```javascript
import { useNavigate } from 'react-router-dom';
import { useToast } from '@/hooks/use-toast';
import FaceCapture from '@/components/FaceCapture';
import { BrandLogo } from '@/components/BrandLogo';

export default function FaceCapturePage() {
  const navigate = useNavigate();
  const { toast } = useToast();

  const handleValidCapture = (base64Image: string) => {
    // TODO: send base64Image to your backend for skin analysis
    toast({
      title: 'Photo captured',
      description: 'Checks passed — sending photo for analysis.',
    });
    console.log('Captured image length:', base64Image.length);
  };

  return (
    <div className="min-h-screen gradient-skinwise-subtle py-8 px-4 relative texture-grain">
      <div className="relative max-w-2xl mx-auto">
        {/* Header */}
        <div className="flex justify-center mb-10 cursor-pointer" onClick={() => navigate('/')}>
          <BrandLogo className="h-10 w-auto" />
        </div>
        <FaceCapture
          onValidCapture={handleValidCapture}
          onCancel={() => navigate(-1)}
        />
      </div>
    </div>
  );
}
```
1. Wraps `<FaceCapture>` inside a responsive card layout.
2. In `handleValidCapture`, logs the base64 character string length to the console and dispatches a toast confirmation.
3. Provides `onCancel={() => navigate(-1)}` to pop the history stack and return to the previous page.

* **Concepts a beginner might not know:**
  * **Delta Navigation (`navigate(-1)`):** Calling `navigate(-1)` in React Router instructs the browser history stack to travel back one step, equivalent to the user clicking their browser's Back button.
  * **Diagnostic Route:** A standalone route specifically maintained during development to test complex hardware interactions (like camera permissions and WebGL inference) without having to fill out a 5-step quiz every time.

* **Why this approach, not an alternative:**
  * *Notice the comment on line 11 (`// TODO: send base64Image to your backend for skin analysis`):* This reveals that `FaceCapturePage` is an isolated sandbox or preview route. The primary production path for face analysis is integrated directly into Step 6 of the Quiz wizard (`src/components/quiz/StepFacePhoto.jsx`), which passes the captured photo to `Results.jsx`.

* **Connects to:**
  * **Imports from:** [FaceCapture.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/FaceCapture.jsx), [BrandLogo.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/BrandLogo.jsx), and [use-toast.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/hooks/use-toast.js).
  * **Referenced by:** [App.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/App.jsx) (bound to `/face-capture`).

---

### `src/hooks/useAssessment.js`

* **What it does:** Encapsulates the form state and validation logic for the 6-step assessment wizard. It maintains the working `SkinAssessment` data model, tracks the active step index, toggles array-based multi-select fields, and evaluates step completion criteria.

* **Full walkthrough:**
```javascript
import { useState, useCallback } from 'react';
import { SkinAssessment } from '@/types/skincare';

const INITIAL_ASSESSMENT: SkinAssessment = {
  ageRange: '',
  gender: '',
  skinType: '',
  skinConcerns: [],
  climate: '',
  allergies: [],
  skinGoals: '',
  budgetRange: '',
};

export function useAssessment() {
  const [assessment, setAssessment] = useState<SkinAssessment>(INITIAL_ASSESSMENT);
  const [currentStep, setCurrentStep] = useState(1);
  const totalSteps = 6;
```
1. **Initial State Blueprint:** Defines `INITIAL_ASSESSMENT` with empty strings and empty arrays for `skinConcerns` and `allergies`. Total steps is fixed at `6`.

```javascript
  const updateAssessment = useCallback((field: keyof SkinAssessment, value: string | string[]) => {
    setAssessment(prev => ({ ...prev, [field]: value }));
  }, []);

  const toggleArrayField = useCallback((field: 'skinConcerns' | 'allergies', value: string) => {
    setAssessment(prev => {
      const current = prev[field];
      const updated = current.includes(value)
        ? current.filter(v => v !== value)
        : [...current, value];
      return { ...prev, [field]: updated };
    });
  }, []);

  const nextStep = useCallback(() => {
    if (currentStep < totalSteps) {
      setCurrentStep(prev => prev + 1);
    }
  }, [currentStep, totalSteps]);

  const prevStep = useCallback(() => {
    if (currentStep > 1) {
      setCurrentStep(prev => prev - 1);
    }
  }, [currentStep]);

  const goToStep = useCallback((step: number) => {
    if (step >= 1 && step <= totalSteps) {
      setCurrentStep(step);
    }
  }, [totalSteps]);

  const resetAssessment = useCallback(() => {
    setAssessment(INITIAL_ASSESSMENT);
    setCurrentStep(1);
  }, []);
```
2. **Immutability & Set Toggling:**
   * `updateAssessment` uses dynamic object property keys `[field]: value`.
   * `toggleArrayField` checks if the value is present: if already selected, it uses `.filter()` to remove it; if not selected, it uses the spread operator `[...current, value]` to append it, returning a new array reference to trigger React re-renders.
   * `nextStep`, `prevStep`, and `goToStep` enforce boundary guards (`1 <= step <= 6`).

```javascript
  const isStepValid = useCallback((step: number): boolean => {
    switch (step) {
      case 1:
        return !!assessment.ageRange;
      case 2:
        return !!assessment.skinType;
      case 3:
        return assessment.skinConcerns.length > 0;
      case 4:
        return !!assessment.climate;
      case 5:
        return !!assessment.budgetRange;
      case 6:
        return true; // Face photo is optional
      default:
        return false;
    }
  }, [assessment]);

  const canProceed = isStepValid(currentStep);

  return {
    assessment,
    currentStep,
    totalSteps,
    updateAssessment,
    toggleArrayField,
    nextStep,
    prevStep,
    goToStep,
    resetAssessment,
    canProceed,
    isStepValid,
  };
}
```
3. **Step-by-Step Validation Gate:**
   * Step 1: Requires `ageRange` (truthy string). Gender is optional.
   * Step 2: Requires `skinType`.
   * Step 3: Requires at least one concern (`skinConcerns.length > 0`).
   * Step 4: Requires `climate`. Allergies and free-form goals are optional.
   * Step 5: Requires `budgetRange`.
   * Step 6: Always valid (`return true`) because the face photo is completely optional.

* **Concepts a beginner might not know:**
  * **Custom React Hook:** A JavaScript function starting with `use` that calls other built-in React hooks (`useState`, `useCallback`) to package reusable stateful logic separately from UI JSX.
  * **`useCallback`:** A React performance optimization hook that caches a function definition between renders, preventing unnecessary child re-renders when passing callbacks down as props.
  * **Double Negation (`!!`):** Converts any truthy or falsy value into a strict boolean (e.g., `!!""` becomes `false`, `!!"oily"` becomes `true`).

* **Why this approach, not an alternative:**
  * *Local Hook State vs. Global Form Library (e.g., React Hook Form / Formik):* While `package.json` installs `react-hook-form`, the author chose a custom hook for the quiz. For multi-step wizard navigation where steps are conditionally unmounted, custom state avoids complex multi-page form register/unregister lifecycles.
  * *Critique / Defensibility:* Form state is currently held purely in React memory. If a user on mobile accidentally refreshes the browser on Step 5, their answers are lost. A stronger implementation would persist `assessment` into `sessionStorage`.

* **Connects to:**
  * **Imports from:** [skincare.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/types/skincare.js).
  * **Imported by:** [Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx).

---

### `src/components/quiz/ProgressIndicator.jsx`

* **What it does:** Renders a horizontal step-indicator widget above the quiz cards, displaying numbered bubbles, connecting lines, step labels, and animated checkmarks for completed stages.

* **Full walkthrough:**
```javascript
import { cn } from '@/lib/utils';
import { Check } from 'lucide-react';

interface ProgressIndicatorProps {
  currentStep: number;
  totalSteps: number;
  isStepValid: (step: number) => boolean;
}

const STEP_LABELS = ['Basic Info', 'Skin Type', 'Concerns', 'Lifestyle', 'Preferences'];

export function ProgressIndicator({ currentStep, totalSteps, isStepValid }: ProgressIndicatorProps) {
  return (
    <div className="w-full mb-8">
      <div className="flex items-center justify-between">
        {Array.from({ length: totalSteps }, (_, i) => i + 1).map((step) => {
          const isComplete = step < currentStep && isStepValid(step);
          const isCurrent = step === currentStep;
          
          return (
            <div key={step} className="flex items-center flex-1 last:flex-none">
              <div className="flex flex-col items-center">
                <div
                  className={cn(
                    'w-10 h-10 rounded-full flex items-center justify-center text-sm font-medium transition-all duration-300',
                    isComplete && 'bg-primary text-primary-foreground',
                    isCurrent && 'bg-primary text-primary-foreground ring-4 ring-primary/20',
                    !isComplete && !isCurrent && 'bg-muted text-muted-foreground'
                  )}
                >
                  {isComplete ? <Check className="w-5 h-5" /> : step}
                </div>
                <span
                  className={cn(
                    'mt-2 text-xs font-medium hidden sm:block',
                    (isComplete || isCurrent) ? 'text-foreground' : 'text-muted-foreground'
                  )}
                >
                  {STEP_LABELS[step - 1]}
                </span>
              </div>
              
              {step < totalSteps && (
                <div
                  className={cn(
                    'flex-1 h-1 mx-2 rounded-full transition-all duration-300',
                    step < currentStep ? 'bg-primary' : 'bg-muted'
                  )}
                />
              )}
            </div>
          );
        })}
      </div>
    </div>
  );
}
```
1. Generates step indices `[1, 2, 3, 4, 5, 6]` dynamically using `Array.from({ length: totalSteps }, (_, i) => i + 1)`.
2. Computes two boolean states for each bubble:
   * `isComplete`: The user has already passed this step AND it satisfies validation. Renders a Lucide `<Check className="w-5 h-5" />`.
   * `isCurrent`: The user is currently answering this step. Adds a pulsing ring (`ring-4 ring-primary/20`).
3. Renders connecting horizontal bar dividers between bubbles (`step < totalSteps`), colored with `bg-primary` for completed tracks and `bg-muted` for unvisited ones.

* **Concepts a beginner might not know:**
  * **`Array.from({ length: N })`:** A standard JavaScript utility that constructs a new array of length $N$, commonly paired with a mapping function to generate sequential numeric ranges without traditional `for` loops.
  * **Tailwind `last:flex-none`:** A pseudo-class modifier applying styles only to the last child in a list, preventing the final step bubble from expanding its flexbox boundary past the right edge.

* **Why this approach, not an alternative:**
  * *Notice an edge case on line 10:* `STEP_LABELS` contains only 5 labels (`['Basic Info', 'Skin Type', 'Concerns', 'Lifestyle', 'Preferences']`), whereas `totalSteps` is 6 (Step 6 is Photo). For step 6, `STEP_LABELS[5]` evaluates to `undefined`. While the text simply renders blank on Step 6 without throwing errors, a clean refactor would expand `STEP_LABELS` to include `'Face Photo'`.

* **Connects to:**
  * **Imports from:** [utils.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/lib/utils.js) and `lucide-react`.
  * **Imported by:** [Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx).

---

### `src/components/quiz/StepBasicInfo.jsx`

* **What it does:** Renders Step 1 of the quiz wizard. Collects age range and optional gender demographics using custom-styled radio buttons.

* **Full walkthrough:**
```javascript
import { SkinAssessment, AGE_RANGES } from '@/types/skincare';
import { Label } from '@/components/ui/label';
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group';
import { Input } from '@/components/ui/input';
import { cn } from '@/lib/utils';

interface StepBasicInfoProps {
  assessment: SkinAssessment;
  onUpdate: (field: keyof SkinAssessment, value: string) => void;
}

export function StepBasicInfo({ assessment, onUpdate }: StepBasicInfoProps) {
  return (
    <div className="space-y-8 animate-fade-in">
      <div className="text-center space-y-2">
        <h2 className="text-2xl font-display font-semibold">Let's Get Started</h2>
        <p className="text-muted-foreground">Tell us a bit about yourself</p>
      </div>

      <div className="space-y-6">
        <div className="space-y-3">
          <Label className="text-base">What's your age range?</Label>
          <RadioGroup
            value={assessment.ageRange}
            onValueChange={(value) => onUpdate('ageRange', value)}
            className="grid grid-cols-2 sm:grid-cols-3 gap-3"
          >
            {AGE_RANGES.map((range) => (
              <div key={range.value}>
                <RadioGroupItem
                  value={range.value}
                  id={range.value}
                  className="peer sr-only"
                />
                <Label
                  htmlFor={range.value}
                  className={cn(
                    'flex items-center justify-center p-4 rounded-lg border-2 cursor-pointer transition-all',
                    'hover:border-primary/50 hover:bg-primary/5',
                    assessment.ageRange === range.value
                      ? 'border-primary bg-primary/10'
                      : 'border-border'
                  )}
                >
                  {range.label}
                </Label>
              </div>
            ))}
          </RadioGroup>
        </div>

        <div className="space-y-3">
          <Label className="text-base">Gender (optional)</Label>
          <RadioGroup
            value={assessment.gender}
            onValueChange={(value) => onUpdate('gender', value)}
            className="grid grid-cols-3 gap-3"
          >
            {[
              { value: 'female', label: 'Female' },
              { value: 'male', label: 'Male' },
              { value: 'other', label: 'Other' },
            ].map((option) => (
              <div key={option.value}>
                <RadioGroupItem
                  value={option.value}
                  id={`gender-${option.value}`}
                  className="peer sr-only"
                />
                <Label
                  htmlFor={`gender-${option.value}`}
                  className={cn(
                    'flex items-center justify-center p-4 rounded-lg border-2 cursor-pointer transition-all',
                    'hover:border-primary/50 hover:bg-primary/5',
                    assessment.gender === option.value
                      ? 'border-primary bg-primary/10'
                      : 'border-border'
                  )}
                >
                  {option.label}
                </Label>
              </div>
            ))}
          </RadioGroup>
        </div>
      </div>
    </div>
  );
}
```
1. Reads `AGE_RANGES` from `skincare.js` and renders a 2-to-3 column grid.
2. **Accessible Custom Radios (`peer sr-only`):** The actual HTML `<input type="radio">` is rendered with `className="peer sr-only"`. This hides the default browser circle from sighted users while keeping it completely visible to screen readers. The sibling `<Label>` acts as the clickable tile, highlighting with `border-primary bg-primary/10` when selected.
3. Renders a second radio group for gender (`female`, `male`, `other`).

* **Concepts a beginner might not know:**
  * **Screen Reader Only (`sr-only`):** A CSS utility that visually hides an element off-screen while preserving it in the DOM accessibility tree so assistive screen reader software can read and announce it.
  * **Radix `RadioGroup`:** An accessible headless component managing arrow key navigation, focus rings, and ARIA attributes for radio lists.

* **Why this approach, not an alternative:**
  * *Tile Cards vs. Native Dropdown Select (`<select>`):* On mobile touchscreens, large clickable tiles have vastly higher usability and lower friction than navigating tiny native dropdown select menus.

* **Connects to:**
  * **Imports from:** [skincare.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/types/skincare.js), [utils.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/lib/utils.js), and shadcn UI primitives.
  * **Imported by:** [Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx).

---

### `src/components/quiz/StepSkinType.jsx`

* **What it does:** Renders Step 2 of the wizard. Presents the five primary dermatological skin types (Oily, Dry, Combination, Sensitive, Normal) with customized illustrative iconography.

* **Full walkthrough:**
```javascript
import { SkinAssessment, SKIN_TYPES } from '@/types/skincare';
import { Label } from '@/components/ui/label';
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group';
import { cn } from '@/lib/utils';
import { Droplets, Sun, Heart, Sparkles, LayoutGrid } from 'lucide-react';

interface StepSkinTypeProps {
  assessment: SkinAssessment;
  onUpdate: (field: keyof SkinAssessment, value: string) => void;
}

const SKIN_TYPE_ICONS = {
  oily: Droplets,
  dry: Sun,
  combination: LayoutGrid,
  sensitive: Heart,
  normal: Sparkles,
};

export function StepSkinType({ assessment, onUpdate }: StepSkinTypeProps) {
  return (
    <div className="space-y-8 animate-fade-in">
      <div className="text-center space-y-2">
        <h2 className="text-2xl font-display font-semibold">Know Your Skin Type</h2>
        <p className="text-muted-foreground">Select the option that best describes your skin</p>
      </div>

      <RadioGroup
        value={assessment.skinType}
        onValueChange={(value) => onUpdate('skinType', value)}
        className="grid gap-4"
      >
        {SKIN_TYPES.map((type) => {
          const Icon = SKIN_TYPE_ICONS[type.value as keyof typeof SKIN_TYPE_ICONS];
          
          return (
            <div key={type.value}>
              <RadioGroupItem
                value={type.value}
                id={`skin-${type.value}`}
                className="peer sr-only"
              />
              <Label
                htmlFor={`skin-${type.value}`}
                className={cn(
                  'flex items-start gap-4 p-4 rounded-lg border-2 cursor-pointer transition-all',
                  'hover:border-primary/50 hover:bg-primary/5',
                  assessment.skinType === type.value
                    ? 'border-primary bg-primary/10'
                    : 'border-border'
                )}
              >
                <div className={cn(
                  'p-2 rounded-lg',
                  assessment.skinType === type.value
                    ? 'bg-primary text-primary-foreground'
                    : 'bg-muted text-muted-foreground'
                )}>
                  <Icon className="w-5 h-5" />
                </div>
                <div className="flex-1">
                  <div className="font-medium">{type.label}</div>
                  <div className="text-sm text-muted-foreground">{type.description}</div>
                </div>
              </Label>
            </div>
          );
        })}
      </RadioGroup>
    </div>
  );
}
```
1. Maps dermatological categories to visual metaphor icons: `Droplets` for Oily, `Sun` for Dry, `LayoutGrid` for Combination (T-zone split), `Heart` for Sensitive, and `Sparkles` for Normal.
2. Renders descriptive text alongside labels (e.g., "Oily: Shiny, enlarged pores, prone to acne") so users without formal skincare knowledge can accurately identify their type.

* **Concepts a beginner might not know:**
  * **Component Record Mapping (`SKIN_TYPE_ICONS`):** An object mapping string enum keys to React component references, allowing dynamic instantiation of icons via `<Icon className="..." />`.
  * **`keyof typeof` Type Indexing:** TypeScript pattern extracting the union of keys from an existing JavaScript object literal (`'oily' | 'dry' | 'combination' | 'sensitive' | 'normal'`).

* **Why this approach, not an alternative:**
  * *Detailed Descriptions vs. Bare Labels:* Many users confuse dehydrated skin with oily skin (dehydrated skin often overproduces oil to compensate). Providing descriptive symptom summaries on the card prevents user misclassification.

* **Connects to:**
  * **Imports from:** [skincare.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/types/skincare.js) and `lucide-react`.
  * **Imported by:** [Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx).

---

### `src/components/quiz/StepConcerns.jsx`

* **What it does:** Renders Step 3 of the wizard. Facilitates multi-selection of skincare concerns (Acne, Pigmentation, Aging, Redness, etc.) with dynamic selection counters.

* **Full walkthrough:**
```javascript
import { SkinAssessment, SKIN_CONCERNS } from '@/types/skincare';
import { Label } from '@/components/ui/label';
import { Checkbox } from '@/components/ui/checkbox';
import { cn } from '@/lib/utils';

interface StepConcernsProps {
  assessment: SkinAssessment;
  onToggle: (field: 'skinConcerns' | 'allergies', value: string) => void;
}

export function StepConcerns({ assessment, onToggle }: StepConcernsProps) {
  return (
    <div className="space-y-8 animate-fade-in">
      <div className="text-center space-y-2">
        <h2 className="text-2xl font-display font-semibold">What Are Your Concerns?</h2>
        <p className="text-muted-foreground">Select all that apply to you</p>
      </div>

      <div className="grid grid-cols-2 gap-3">
        {SKIN_CONCERNS.map((concern) => {
          const isSelected = assessment.skinConcerns.includes(concern.value);
          
          return (
            <div
              key={concern.value}
              onClick={() => onToggle('skinConcerns', concern.value)}
              className={cn(
                'flex items-center gap-3 p-4 rounded-lg border-2 cursor-pointer transition-all',
                'hover:border-primary/50 hover:bg-primary/5',
                isSelected
                  ? 'border-primary bg-primary/10'
                  : 'border-border'
              )}
            >
              <Checkbox
                checked={isSelected}
                onCheckedChange={() => onToggle('skinConcerns', concern.value)}
                className="pointer-events-none"
              />
              <Label className="cursor-pointer text-sm font-medium">
                {concern.label}
              </Label>
            </div>
          );
        })}
      </div>

      {assessment.skinConcerns.length > 0 && (
        <p className="text-center text-sm text-muted-foreground">
          {assessment.skinConcerns.length} concern{assessment.skinConcerns.length !== 1 && 's'} selected
        </p>
      )}
    </div>
  );
}
```
1. Iterates over `SKIN_CONCERNS` to build a 2-column grid.
2. Attaches the `onClick` handler directly to the parent tile `<div>`. Notice the nested `<Checkbox className="pointer-events-none" />`: by disabling pointer events on the checkbox itself, clicking anywhere on the entire card invokes `onToggle` cleanly without double-triggering event propagation.
3. Renders a dynamic pluralized footer count (`X concerns selected`).

* **Concepts a beginner might not know:**
  * **Event Propagation / Double Bubbling:** In nested clickable elements, clicking a child checkbox inside a clickable parent div triggers two click events. Marking the inner child as `pointer-events-none` routes the click event exclusively through the parent tile.
  * **Pluralization Logic:** Conditionally appending `'s'` (`length !== 1 && 's'`) ensures grammatical correctness ("1 concern" vs. "2 concerns").

* **Why this approach, not an alternative:**
  * *Card-Level Click Targets:* Users frequently miss small 16px checkboxes on mobile phones. Expanding the click target to the full rectangular card dramatically reduces tap error rates.

* **Connects to:**
  * **Imports from:** [skincare.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/types/skincare.js) and [utils.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/lib/utils.js).
  * **Imported by:** [Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx).

---

### `src/components/quiz/StepLifestyle.jsx`

* **What it does:** Renders Step 4 of the wizard. Gathers environmental factors (climate), ingredient allergies/sensitivities, and an optional open-ended text description of the user's specific skin goals.

* **Full walkthrough:**
```javascript
import { SkinAssessment, CLIMATES, COMMON_ALLERGIES } from '@/types/skincare';
import { Label } from '@/components/ui/label';
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group';
import { Checkbox } from '@/components/ui/checkbox';
import { Textarea } from '@/components/ui/textarea';
import { cn } from '@/lib/utils';

interface StepLifestyleProps {
  assessment: SkinAssessment;
  onUpdate: (field: keyof SkinAssessment, value: string) => void;
  onToggle: (field: 'skinConcerns' | 'allergies', value: string) => void;
}

export function StepLifestyle({ assessment, onUpdate, onToggle }: StepLifestyleProps) {
  return (
    <div className="space-y-8 animate-fade-in">
      <div className="text-center space-y-2">
        <h2 className="text-2xl font-display font-semibold">Your Environment</h2>
        <p className="text-muted-foreground">Help us understand your lifestyle factors</p>
      </div>

      <div className="space-y-6">
        <div className="space-y-3">
          <Label className="text-base">What's your climate like?</Label>
          <RadioGroup
            value={assessment.climate}
            onValueChange={(value) => onUpdate('climate', value)}
            className="grid gap-2"
          >
            {CLIMATES.map((climate) => (
              <div key={climate.value}>
                <RadioGroupItem
                  value={climate.value}
                  id={`climate-${climate.value}`}
                  className="peer sr-only"
                />
                <Label
                  htmlFor={`climate-${climate.value}`}
                  className={cn(
                    'flex items-center justify-between p-4 rounded-lg border-2 cursor-pointer transition-all',
                    'hover:border-primary/50 hover:bg-primary/5',
                    assessment.climate === climate.value
                      ? 'border-primary bg-primary/10'
                      : 'border-border'
                  )}
                >
                  {climate.label}
                </Label>
              </div>
            ))}
          </RadioGroup>
        </div>

        <div className="space-y-3">
          <Label className="text-base">Any ingredient sensitivities? (optional)</Label>
          <div className="grid grid-cols-2 gap-2">
            {COMMON_ALLERGIES.map((allergy) => {
              const isSelected = assessment.allergies.includes(allergy.value);
              
              return (
                <div
                  key={allergy.value}
                  onClick={() => onToggle('allergies', allergy.value)}
                  className={cn(
                    'flex items-center gap-2 p-3 rounded-lg border cursor-pointer transition-all',
                    'hover:border-primary/50 hover:bg-primary/5',
                    isSelected
                      ? 'border-primary bg-primary/10'
                      : 'border-border'
                  )}
                >
                  <Checkbox
                    checked={isSelected}
                    onCheckedChange={() => onToggle('allergies', allergy.value)}
                    className="pointer-events-none"
                  />
                  <span className="text-sm">{allergy.label}</span>
                </div>
              );
            })}
          </div>
        </div>

        <div className="space-y-3">
          <Label className="text-base">What are your skincare goals? (optional)</Label>
          <Textarea
            placeholder="E.g., I want to reduce acne scars and have a glowing complexion..."
            value={assessment.skinGoals}
            onChange={(e) => onUpdate('skinGoals', e.target.value)}
            className="min-h-[100px] resize-none"
          />
        </div>
      </div>
    </div>
  );
}
```
1. Climate Radio List: Maps `CLIMATES` (Tropical, Dry, Temperate, Cold, Varied).
2. Allergies Checklist: Multi-select array toggling common sensitizers (Fragrance, Parabens, Sulfates, Retinol, Salicylic Acid, Niacinamide).
3. Free-form Textarea: Captures qualitative user goals with a locked height (`resize-none`).

* **Concepts a beginner might not know:**
  * **Environmental Skincare Context:** In tropical humid Indian climates, heavy occlusive creams cause fungal acne and blackheads; lightweight gels are required. In arid winter zones, humectants require occlusive sealants. Collecting climate data enables the AI model to adjust formulation viscosity.
  * **`resize-none`:** CSS property that prevents users from manually dragging and resizing a `<textarea>`, preventing layout distortion in responsive card containers.

* **Why this approach, not an alternative:**
  * *Predefined Allergens + Free-form Goals:* Combining structured checkboxes for known irritants with an open-ended goals textarea gives the LLM both strict negative constraints (ingredients to strictly avoid) and subjective creative context (user desires).

* **Connects to:**
  * **Imports from:** [skincare.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/types/skincare.js), [utils.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/lib/utils.js), and shadcn UI primitives.
  * **Imported by:** [Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx).

---

### `src/components/quiz/StepPreferences.jsx`

* **What it does:** Renders Step 5 of the wizard. Guides the user to choose an Indian Rupee (`₹`) budget tier (Budget, Mid-Range, Premium, Luxury) to ground product recommendations in their purchasing power.

* **Full walkthrough:**
```javascript
import { SkinAssessment, BUDGET_RANGES } from '@/types/skincare';
import { Label } from '@/components/ui/label';
import { RadioGroup, RadioGroupItem } from '@/components/ui/radio-group';
import { cn } from '@/lib/utils';
import { Wallet, CreditCard, Gem, Crown } from 'lucide-react';

interface StepPreferencesProps {
  assessment: SkinAssessment;
  onUpdate: (field: keyof SkinAssessment, value: string) => void;
}

const BUDGET_ICONS = {
  budget: Wallet,
  mid: CreditCard,
  premium: Gem,
  luxury: Crown,
};

export function StepPreferences({ assessment, onUpdate }: StepPreferencesProps) {
  return (
    <div className="space-y-8 animate-fade-in">
      <div className="text-center space-y-2">
        <h2 className="text-2xl font-display font-semibold">Almost There!</h2>
        <p className="text-muted-foreground">Let us know your budget preferences</p>
      </div>

      <RadioGroup
        value={assessment.budgetRange}
        onValueChange={(value) => onUpdate('budgetRange', value)}
        className="grid gap-4"
      >
        {BUDGET_RANGES.map((budget) => {
          const Icon = BUDGET_ICONS[budget.value as keyof typeof BUDGET_ICONS];
          
          return (
            <div key={budget.value}>
              <RadioGroupItem
                value={budget.value}
                id={`budget-${budget.value}`}
                className="peer sr-only"
              />
              <Label
                htmlFor={`budget-${budget.value}`}
                className={cn(
                  'flex items-center gap-4 p-5 rounded-lg border-2 cursor-pointer transition-all',
                  'hover:border-primary/50 hover:bg-primary/5',
                  assessment.budgetRange === budget.value
                    ? 'border-primary bg-primary/10'
                    : 'border-border'
                )}
              >
                <div className={cn(
                  'p-3 rounded-lg',
                  assessment.budgetRange === budget.value
                    ? 'bg-primary text-primary-foreground'
                    : 'bg-muted text-muted-foreground'
                )}>
                  <Icon className="w-6 h-6" />
                </div>
                <div className="flex-1">
                  <div className="font-medium text-lg">{budget.label}</div>
                </div>
              </Label>
            </div>
          );
        })}
      </RadioGroup>
    </div>
  );
}
```
1. Maps budget tiers to visual icons: `Wallet` for Budget (Under ₹500), `CreditCard` for Mid-Range (₹500–₹1500), `Gem` for Premium (₹1500–₹3000), and `Crown` for Luxury (₹3000+).
2. Updates `assessment.budgetRange` on selection.

* **Concepts a beginner might not know:**
  * **Price Grounding in LLM Engineering:** Generative AI models frequently recommend luxury imported brands (e.g., Drunk Elephant, SkinCeuticals costing ₹8,000+) unless bound to explicit numerical price ceilings in their system prompt. This step provides the numerical boundaries passed directly into Gemini.

* **Why this approach, not an alternative:**
  * *Categorical Tiers vs. Free-form Price Slider:* A slider from ₹100 to ₹10,000 adds unnecessary precision friction. Skincare products naturally cluster in standard commercial retail tiers in India (drugstore/mass market <₹500; D2C brands like Minimalist/Plum ₹500–₹1500; prestige brands ₹1500–₹3000).

* **Connects to:**
  * **Imports from:** [skincare.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/types/skincare.js) and `lucide-react`.
  * **Imported by:** [Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx).

---

### `src/components/quiz/StepFacePhoto.jsx`

* **What it does:** Renders Step 6 (the final step) of the wizard. Provides an optional invitation to capture a live face photo for multimodal AI analysis, with toggling between the prompt card and `<FaceCapture>`.

* **Full walkthrough:**
```javascript
import { useState } from 'react';
import { Button } from '@/components/ui/button';
import { Camera, SkipForward, CheckCircle2 } from 'lucide-react';
import FaceCapture from '@/components/FaceCapture';

interface StepFacePhotoProps {
  onCapture: (base64: string) => void;
  captured: boolean;
}

export function StepFacePhoto({ onCapture, captured }: StepFacePhotoProps) {
  const [showCamera, setShowCamera] = useState(false);

  if (showCamera) {
    return (
      <div className="animate-fade-in">
        <FaceCapture
          onValidCapture={(img) => {
            onCapture(img);
            setShowCamera(false);
          }}
          onCancel={() => setShowCamera(false)}
        />
      </div>
    );
  }

  return (
    <div className="space-y-8 animate-fade-in">
      <div className="text-center space-y-2">
        <h2 className="text-2xl font-display font-semibold">Face Photo (Optional)</h2>
        <p className="text-muted-foreground">
          Take a photo for more accurate, AI-powered skin analysis. You can skip this step.
        </p>
      </div>

      <div className="flex flex-col items-center gap-4 py-6">
        {captured ? (
          <div className="flex items-center gap-2 text-primary">
            <CheckCircle2 className="w-5 h-5" />
            <span className="font-medium">Photo captured successfully</span>
          </div>
        ) : null}

        <Button onClick={() => setShowCamera(true)} className="gap-2" size="lg">
          <Camera className="w-5 h-5" />
          {captured ? 'Retake Photo' : 'Take Face Photo'}
        </Button>
      </div>
    </div>
  );
}
```
1. `showCamera` toggles between the introductory explanation card and the full `<FaceCapture>` viewfinder.
2. When the user successfully captures a verified photo, `onValidCapture` passes the base64 JPEG string up to `Quiz.jsx` and toggles `showCamera` back to `false`.
3. If already captured, displays a green success confirmation `<CheckCircle2 />` and switches the button label to `'Retake Photo'`.

* **Concepts a beginner might not know:**
  * **Conditional View Switching:** Swapping between different UI modes based on component state (`if (showCamera) return <FaceCapture ... />`) without requiring a separate URL route.
  * **Multimodal AI Inputs:** Providing both structured text prompts (the survey answers) and raw binary visual data (the face JPEG) into an AI model simultaneously so it can correlate user assertions with visual evidence.

* **Why this approach, not an alternative:**
  * *Making Photo Capture Strictly Optional:* Forcing a user to take a webcam photo causes massive drop-off rates due to privacy concerns or environmental limitations (e.g., user in a dark room or public place). Making it explicitly optional ensures the user can complete their consultation regardless.

* **Connects to:**
  * **Imports from:** [FaceCapture.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/FaceCapture.jsx) and `lucide-react`.
  * **Imported by:** [Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx).

---

### `src/pages/Quiz.jsx`

* **What it does:** The primary assessment page controller. It coordinates the 6 step components, displays the top progress bar and navigation buttons, inserts the completed assessment row into Supabase PostgreSQL, and transitions the user to the results page.

* **Full walkthrough:**
```javascript
import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '@/contexts/AuthContext';
import { useAssessment } from '@/hooks/useAssessment';
import { ProgressIndicator } from '@/components/quiz/ProgressIndicator';
import { StepBasicInfo } from '@/components/quiz/StepBasicInfo';
import { StepSkinType } from '@/components/quiz/StepSkinType';
import { StepConcerns } from '@/components/quiz/StepConcerns';
import { StepLifestyle } from '@/components/quiz/StepLifestyle';
import { StepPreferences } from '@/components/quiz/StepPreferences';
import { StepFacePhoto } from '@/components/quiz/StepFacePhoto';
import { Button } from '@/components/ui/button';
import { Card, CardContent } from '@/components/ui/card';
import { useToast } from '@/hooks/use-toast';
import { supabase } from '@/integrations/supabase/client';
import { ArrowLeft, ArrowRight, Sparkles, Loader2 } from 'lucide-react';
import { BrandLogo } from '@/components/BrandLogo';

export default function Quiz() {
  const { user } = useAuth();
  const navigate = useNavigate();
  const { toast } = useToast();
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [facePhoto, setFacePhoto] = useState<string | null>(null);
  
  const {
    assessment,
    currentStep,
    totalSteps,
    updateAssessment,
    toggleArrayField,
    nextStep,
    prevStep,
    canProceed,
    isStepValid,
  } = useAssessment();
```
1. Initializes `useAssessment()` to retrieve step controls, validation flags, and form state. Maintains local state for `facePhoto` (base64 string) and `isSubmitting`.

```javascript
  const handleSubmit = async () => {
    setIsSubmitting(true);
    try {
      let finalId = `mock-${Date.now()}`;

      // Only save to Supabase if the user is logged in
      if (user) {
        const { data, error } = await supabase
          .from('skin_assessments')
          .insert([
            {
              user_id: user.id,
              skin_type: assessment.skinType,
              skin_concerns: assessment.skinConcerns,
              age_range: assessment.ageRange,
              climate: assessment.climate,
              budget_range: assessment.budgetRange,
            },
          ])
          .select()
          .single();

        if (error) throw error;
        if (data) finalId = data.id;
      } else {
        // Fallback for demonstration if user is not logged in
        await new Promise(resolve => setTimeout(resolve, 800));
      }

      // Navigate to results with the actual or mock ID
      navigate(`/results/${finalId}`, {
        state: { facePhoto, assessment },
      });
    } catch (error) {
      console.error('Error saving assessment:', error);
      toast({
        title: 'Error',
        description: 'Failed to save your assessment. Please try again.',
        variant: 'destructive',
      });
    } finally {
      setIsSubmitting(false);
    }
  };
```
2. **Submission & Database Write:**
   * Maps camelCase hook properties (`assessment.skinType`) to PostgreSQL snake_case database columns (`skin_type`, `skin_concerns`, `age_range`, `climate`, `budget_range`).
   * Executes `supabase.from('skin_assessments').insert([...]).select().single()`.
   * Captures the newly generated UUID `data.id`.
   * Invokes `navigate(/results/${finalId}, { state: { facePhoto, assessment } })`. Note that `facePhoto` is passed via React Router navigation state in memory, avoiding sending raw megabyte-sized image base64 strings into the database table directly!

```javascript
  const renderStep = () => {
    switch (currentStep) {
      case 1:
        return <StepBasicInfo assessment={assessment} onUpdate={updateAssessment} />;
      case 2:
        return <StepSkinType assessment={assessment} onUpdate={updateAssessment} />;
      case 3:
        return <StepConcerns assessment={assessment} onToggle={toggleArrayField} />;
      case 4:
        return <StepLifestyle assessment={assessment} onUpdate={updateAssessment} onToggle={toggleArrayField} />;
      case 5:
        return <StepPreferences assessment={assessment} onUpdate={updateAssessment} />;
      case 6:
        return <StepFacePhoto onCapture={(img) => setFacePhoto(img)} captured={!!facePhoto} />;
      default:
        return null;
    }
  };
```
3. Dynamically renders the active step view via a `switch` statement.
4. Renders navigation controls: Back button is disabled on Step 1; Next button is disabled when `!canProceed`; Step 6 displays `'Get Recommendations'` with an animated spinner during submission.

* **Concepts a beginner might not know:**
  * **Router State (`navigate(path, { state })`):** Passing temporary, non-persisted JavaScript objects between page transitions in browser memory without displaying data in the visible URL query string.
  * **PostgreSQL `.select().single()`:** A PostgREST modifier instructing the database to immediately return the single newly created row representation instead of an empty acknowledgment array.

* **Why this approach, not an alternative:**
  * *Router State vs. Image Upload to Cloud Storage:* The app passes `facePhoto` directly to `Results.jsx` in router memory, which then forwards it to the Edge Function. The photo is never stored in Supabase Storage or an S3 bucket. This ensures maximum biometric privacy: once the Edge Function processes the image with Gemini, the image is discarded and no permanent photo files persist on servers.

* **Connects to:**
  * **Imports from:** [useAssessment.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/hooks/useAssessment.js), [client.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/integrations/supabase/client.js), all 6 quiz step components, and UI primitives.
  * **Routes to:** [Results.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Results.jsx) (`/results/:assessmentId`).

---

### `supabase/functions/get-recommendations/index.js`

* **What it does:** The primary server-side API endpoint for the application, executed on Supabase's Deno-based distributed Edge compute runtime. It authenticates incoming user requests via JWT tokens, constructs a clinically informed prompt bound to Indian budgets and dermatological parameters, transmits the prompt and optional face photo to Google Gemini 2.5 Flash, parses structured JSON recommendations, and enriches each product with direct e-commerce purchase links using the Firecrawl search API.

* **Full walkthrough:**
```javascript
import { serve } from "https://deno.land/std@0.168.0/http/server.ts";
import { createClient } from "https://esm.sh/@supabase/supabase-js@2";

const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type, x-supabase-client-platform, x-supabase-client-platform-version, x-supabase-client-runtime, x-supabase-client-runtime-version',
};

serve(async (req) => {
  if (req.method === 'OPTIONS') {
    return new Response(null, { headers: corsHeaders });
  }
```
1. **Deno Runtime & CORS Preflight:** Runs on Deno (importing dependencies directly from URLs like `esm.sh` and `deno.land`). Responds immediately to HTTP `OPTIONS` preflight requests with open `corsHeaders`, permitting cross-origin requests from the browser frontend.

```javascript
  try {
    // Authenticate the user
    const authHeader = req.headers.get('Authorization');
    if (!authHeader?.startsWith('Bearer ')) {
      return new Response(JSON.stringify({ error: 'Unauthorized' }), {
        status: 401,
        headers: { ...corsHeaders, 'Content-Type': 'application/json' },
      });
    }

    const supabaseClient = createClient(
      Deno.env.get('SUPABASE_URL') ?? '',
      Deno.env.get('SUPABASE_ANON_KEY') ?? '',
      { global: { headers: { Authorization: authHeader } } }
    );

    const token = authHeader.replace('Bearer ', '');
    const { data: { user }, error: authError } = await supabaseClient.auth.getUser(token);
    if (authError || !user) {
      return new Response(JSON.stringify({ error: 'Unauthorized' }), {
        status: 401,
        headers: { ...corsHeaders, 'Content-Type': 'application/json' },
      });
    }

    const userId = user.id;
    console.log('Authenticated user:', userId);
```
2. **Cryptographic JWT Verification:** Extracts the Bearer token from the incoming HTTP `Authorization` header and validates it cryptographically against Supabase's auth engine via `supabaseClient.auth.getUser(token)`. If expired, altered, or missing, execution halts with HTTP `401 Unauthorized`.

```javascript
    const { assessment, facePhoto } = await req.json();
    const GEMINI_API_KEY = Deno.env.get('GEMINI_API_KEY');
    const LOVABLE_API_KEY = Deno.env.get('LOVABLE_API_KEY');
    const FIRECRAWL_API_KEY = Deno.env.get('FIRECRAWL_API_KEY');

    if (!GEMINI_API_KEY && !LOVABLE_API_KEY) {
      console.error('Neither GEMINI_API_KEY nor LOVABLE_API_KEY is configured');
      throw new Error('AI credentials not configured in Edge Function');
    }

    // Build allergy avoidance list
    const allergiesToAvoid = assessment.allergies?.length > 0 
      ? `IMPORTANT: User has allergies to: ${assessment.allergies.join(', ')}. Do NOT recommend products containing these ingredients.`
      : '';

    // Map budget to price expectations
    const budgetMap: Record<string, { description: string; max: number | null; min: number }> = {
      'budget': { description: 'affordable products strictly under ₹500 INR each', max: 500, min: 0 },
      'mid': { description: 'mid-range products between ₹500-₹1500 INR each', max: 1500, min: 500 },
      'premium': { description: 'premium products between ₹1500-₹3000 INR each', max: 3000, min: 1500 },
      'luxury': { description: 'high-end luxury products over ₹3000 INR each', max: null, min: 3000 }
    };
    const budgetInfo = budgetMap[assessment.budget_range] || { description: 'products across various price points', max: null, min: 0 };
    const budgetDescription = budgetInfo.description;
```
3. **Clinical & Economic Guardrails:**
   * Injects strict negative prompts for reported user allergies (`Do NOT recommend products containing these ingredients`).
   * Binds budget strings (`'budget'`, `'mid'`, etc.) to concrete numerical price ranges in Indian Rupees (`min`, `max`, `description`).

```javascript
    const prompt = `You are a skincare expert specializing in Indian skincare products. Analyze this user's skin profile and recommend 5 specific, REAL skincare products available in India.${facePhotoInstruction}

USER PROFILE:
- Age Range: ${assessment.age_range}
- Skin Type: ${assessment.skin_type}
- Skin Concerns: ${assessment.skin_concerns?.join(', ') || 'None specified'}
- Climate: ${assessment.climate || 'Moderate'}
- Goals: ${assessment.skin_goals || 'General skincare improvement'}
- Budget: ${assessment.budget_range} (recommend ${budgetDescription})

${allergiesToAvoid}

REQUIREMENTS:
1. Recommend REAL products from brands available in India like Minimalist, Dot & Key, Plum, Mamaearth, The Derma Co, Cetaphil, Biotique, Lakme, Re'equil, Deconstruct, Fixderma, Episoft, Cipla, etc.
2. CRITICAL BUDGET RULE: Every single product MUST cost ${budgetDescription}. ${budgetInfo.max ? `No product should exceed ₹${budgetInfo.max}.` : ''} ${budgetInfo.min > 0 ? `No product should be below ₹${budgetInfo.min}.` : ''} Double-check each price before including it. If a product is outside the budget range, replace it with a cheaper/appropriate alternative.
3. The "priceRange" field must show the approximate MRP on Indian e-commerce sites (Amazon India, Nykaa, Flipkart). Be as accurate as possible with real current prices.
4. Include a cleanser, moisturizer, and treatments for their specific concerns
5. Each product must be purchasable on Indian e-commerce platforms (Amazon India, Nykaa, Flipkart, etc.)
${facePhoto ? '6. Reference specific visible skin observations from the photo in your summary and product justifications' : ''}

Return ONLY valid JSON in this exact format:
{
  "summary": "A 2-3 sentence personalized summary of why these products suit this user...",
  "products": [
    {
      "name": "Exact Product Name",
      "brand": "Brand Name",
      "description": "What this product does",
      "keyIngredients": ["ingredient1", "ingredient2"],
      "priceRange": "₹XXX",
      "whySuitable": "Why this specific product addresses their skin type and concerns",
      "usageInstructions": "When and how to use",
      "category": "cleanser"
    }
  ]
}`;
```
4. **Structured Prompt Specification:** Demands 5 concrete real products from established Indian brands (Minimalist, Dot & Key, Plum, Derma Co, Cipla), enforces cleanser + moisturizer + treatment category balance, and requires pure JSON output matching a strict schema.

```javascript
    let content = '';

    if (GEMINI_API_KEY) {
      console.log('Calling Google Gemini API directly...');
      const parts: any[] = [{ text: prompt }];

      if (facePhoto && facePhoto.startsWith('data:image/')) {
        const mimeType = facePhoto.substring(5, facePhoto.indexOf(';'));
        const base64Data = facePhoto.substring(facePhoto.indexOf(',') + 1);
        parts.push({
          inlineData: {
            mimeType: mimeType,
            data: base64Data
          }
        });
      }

      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=${GEMINI_API_KEY}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts }],
          generationConfig: {
            responseMimeType: "application/json"
          }
        })
      });

      if (!response.ok) {
        const errorText = await response.text();
        throw new Error(`Gemini API error: ${errorText}`);
      }

      const geminiData = await response.json();
      content = geminiData.candidates?.[0]?.content?.parts?.[0]?.text || '';
    } else {
      // Secondary fallback via Lovable AI Gateway
      ...
    }
```
5. **Multimodal Gemini 2.5 Flash Dispatch:**
   * If `facePhoto` is present, parses the Data URL header to extract `mimeType` (e.g., `image/jpeg`) and strips off the `data:image/jpeg;base64,` prefix.
   * Packages image data using Gemini's native `inlineData: { mimeType, data }` format.
   * Transmits the payload directly to `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent`. Setting `generationConfig.responseMimeType = "application/json"` forces the Gemini engine's decoder to constrain token generation to syntactically valid JSON.

```javascript
    // Clean up markdown code blocks if present
    content = content.replace(/```json\n?/g, '').replace(/```\n?/g, '').trim();
    const result = JSON.parse(content);

    // Enrich products with purchase URLs using Firecrawl
    if (FIRECRAWL_API_KEY) {
      const enrichedProducts = await Promise.all(
        result.products.map(async (product: any) => {
          try {
            const searchQuery = `${product.brand} ${product.name} buy India`;
            const searchResponse = await fetch('https://api.firecrawl.dev/v1/search', {
              method: 'POST',
              headers: {
                'Authorization': `Bearer ${FIRECRAWL_API_KEY}`,
                'Content-Type': 'application/json',
              },
              body: JSON.stringify({ query: searchQuery, limit: 5 }),
            });
            
            if (searchResponse.ok) {
              const searchData = await searchResponse.json();
              const shoppingDomains = ['amazon.in', 'nykaa.com', 'flipkart.com', 'purplle.com', 'myntra.com', 'tatacliq.com', 'netmeds.com', '1mg.com'];
              
              let productUrl = null;
              for (const item of searchData.data || []) {
                const url = item.url?.toLowerCase() || '';
                if (shoppingDomains.some(domain => url.includes(domain))) {
                  productUrl = item.url;
                  break;
                }
              }
              return { ...product, productUrl: productUrl || searchData.data?.[0]?.url };
            }
          } catch (e) {
            console.error(`Firecrawl search failed for ${product.name}:`, e);
          }
          return product;
        })
      );
      result.products = enrichedProducts;
    } else {
      // Fallback to deterministic Amazon India search URL
      result.products = result.products.map((product: any) => ({
        ...product,
        productUrl: `https://www.amazon.in/s?k=${encodeURIComponent(product.brand + ' ' + product.name)}`
      }));
    }

    return new Response(JSON.stringify(result), {
      headers: { ...corsHeaders, 'Content-Type': 'application/json' },
    });
```
6. **E-Commerce URL Resolution & Fallback:**
   * If `FIRECRAWL_API_KEY` is configured, executes parallel searches (`Promise.all`) querying `api.firecrawl.dev/v1/search` for real retail URLs. It filters results to find trusted Indian retailers (`amazon.in`, `nykaa.com`, `flipkart.com`, `purplle.com`, `myntra.com`, `tatacliq.com`, `netmeds.com`, `1mg.com`).
   * **Robust Zero-Config Fallback:** If `FIRECRAWL_API_KEY` is omitted or API calls timeout, it constructs deterministic, pre-encoded Amazon India search queries: `https://www.amazon.in/s?k=${encodeURIComponent(brand + ' ' + name)}`. The frontend never displays a broken button!

* **Concepts a beginner might not know:**
  * **Edge Computing:** Running backend serverless code on globally distributed server nodes situated close to the user geographically (rather than in a single central data center), minimizing round-trip network latency.
  * **CORS (Cross-Origin Resource Sharing):** A browser security protocol that restricts web pages in one domain from making HTTP requests to a different domain unless the destination server explicitly returns permissive `Access-Control-Allow-Origin` headers.
  * **Prompt Engineering (Negative Constraints):** Explicit instructions supplied to an AI model detailing what it must *not* do (e.g., omitting user allergens and filtering out non-Indian retail products).
  * **URL Encoding (`encodeURIComponent`):** Converting spaces and special characters into percent-encoded ASCII values (e.g., spaces become `%20` or `+`) so arbitrary strings can safely form valid HTTP web links.

* **Why this approach, not an alternative:**
  * *Serverless Edge Function vs. Direct Frontend API Calls:* Calling Gemini and Firecrawl directly from the React frontend would expose secret API keys (`GEMINI_API_KEY`, `FIRECRAWL_API_KEY`) to anyone opening browser Developer Tools. Routing requests through the Edge Function keeps API keys securely guarded in server environment variables.
  * *Dual AI Gateway (Direct Gemini vs Lovable Gateway):* Implements high availability: if the direct Google API key is absent, the function switches automatically to the secondary Lovable AI gateway.

* **Connects to:**
  * **Invoked by:** [Results.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Results.jsx) via `supabase.functions.invoke('get-recommendations')`.
  * **Calls external APIs:** Google Gemini 2.5 Flash REST API (`generativelanguage.googleapis.com`) and Firecrawl API (`api.firecrawl.dev`).
  * **Configured in:** [supabase/config.toml](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/supabase/config.toml).

---

### `supabase/config.toml`

* **What it does:** The local and deployment configuration file for the Supabase CLI. Defines the active project reference ID and controls gateway-level settings for Edge Functions.

* **Full walkthrough:**
```toml
project_id = "jplrjkaztkxzpwpeyskc"

[functions.get-recommendations]
verify_jwt = false
```
1. `project_id`: Links the local development directory to the remote Supabase cloud project instance.
2. `verify_jwt = false`: Instructs the Supabase API Gateway not to automatically reject unauthenticated requests at the edge gateway layer.

* **Concepts a beginner might not know:**
  * **TOML (Tom's Obvious, Minimal Language):** A minimalist configuration file format designed to be easy to read due to obvious semantics and key-value mapping.
  * **Gateway vs Application JWT Verification:** In Supabase, setting `verify_jwt = true` causes the gateway proxy (Kong) to reject any request without a valid token before the Edge Function starts executing. Setting `verify_jwt = false` allows the request to reach the function code, allowing custom error handling.

* **Why this approach, not an alternative:**
  * *Why `verify_jwt = false` when authentication is required?* In `get-recommendations/index.js` lines 16–37, the function performs *manual* programmatic JWT verification using `supabaseClient.auth.getUser(token)`. When gateway verification is disabled, unauthorized requests trigger the function's custom JSON error handler (`{ error: 'Unauthorized' }`) with full CORS headers, whereas gateway rejection often results in opaque generic HTML error pages that trigger confusing CORS errors on the frontend!

* **Connects to:**
  * **Interacts with:** Supabase CLI and [get-recommendations/index.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/supabase/functions/get-recommendations/index.js).

---

### `supabase/migrations/20260719111000_add_delete_policies.sql`

* **What it does:** A PostgreSQL SQL database migration script that establishes Row Level Security (RLS) DELETE permissions on the `skin_assessments` and `recommendations` tables.

* **Full walkthrough:**
```sql
-- Add DELETE Row Level Security (RLS) policies for skin_assessments and recommendations

-- For skin_assessments:
CREATE POLICY "Users can delete their own assessments" 
ON public.skin_assessments 
FOR DELETE 
TO authenticated 
USING (auth.uid() = user_id);

-- For recommendations:
CREATE POLICY "Users can delete their own recommendations" 
ON public.recommendations 
FOR DELETE 
TO authenticated 
USING (auth.uid() = user_id);
```
1. Creates two Row Level Security policies using `CREATE POLICY`.
2. Targets `skin_assessments` and `recommendations` tables specifically for `DELETE` operations.
3. Restricts the policy `TO authenticated` users.
4. **Security Predicate (`USING (auth.uid() = user_id)`):** Evaluates the currently authenticated session user's ID (`auth.uid()`). If a user tries to delete a row where `user_id` does not match their own ID, PostgreSQL blocks the query, preventing unauthorized deletion of another user's records.

* **Concepts a beginner might not know:**
  * **Row Level Security (RLS):** A database security feature in PostgreSQL where the database itself evaluates access rules per row, guaranteeing that client applications cannot bypass authorization even if they execute raw `DELETE FROM` statements.
  * **SQL Migration:** A version-controlled script that applies structured schema alterations (creating tables, altering columns, adding policies) to a relational database in a reproducible sequence.

* **Why this approach, not an alternative:**
  * *Database RLS vs Application Layer Security:* Implementing authorization in the database layer guarantees data isolation regardless of bugs or oversights in frontend code. Even if a malicious user alters the client-side JavaScript to request `DELETE FROM skin_assessments WHERE id = 'someone_elses_id'`, Postgres enforces the policy and rejects the query.

* **Connects to:**
  * **Applied to:** PostgreSQL tables `public.skin_assessments` and `public.recommendations`.
  * **Enables deletion in:** [History.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/History.jsx).

---

### `supabase/migrations/20260712_create_heartbeat_table.sql`

* **What it does:** Creates a minimal table and inserts a single keepalive target row to prevent the Supabase project from being automatically paused on the Free tier.

* **Full walkthrough:**
```sql
-- Migration: Create _heartbeat table for the supabase-keepalive workflow.
-- This tiny table exists solely so the GitHub Actions cron job can make a
-- real Postgres read query (via the REST API) to prevent Supabase from
-- auto-pausing the project on the Free plan.  It has no RLS policies,
-- so the service_role key can read it without extra setup.

CREATE TABLE IF NOT EXISTS public._heartbeat (
  id   integer PRIMARY KEY DEFAULT 1,
  note text    NOT NULL    DEFAULT 'keepalive ping target'
);

-- Ensure exactly one row exists for the query to return.
INSERT INTO public._heartbeat (id, note)
VALUES (1, 'keepalive ping target')
ON CONFLICT (id) DO NOTHING;
```
1. Creates table `public._heartbeat` with an `id` integer primary key and a `note` text field.
2. Inserts a single record `(1, 'keepalive ping target')`.
3. `ON CONFLICT (id) DO NOTHING`: Ensures idempotency, so running the migration multiple times never triggers unique key constraint violation errors.

* **Concepts a beginner might not know:**
  * **Idempotency:** An operation that can be executed multiple times without changing the result beyond the initial application (e.g., `IF NOT EXISTS` and `ON CONFLICT DO NOTHING`).
  * **SaaS Free Tier Auto-Pausing:** Cloud platforms like Supabase automatically pause free-tier Postgres database instances after 7 days of inactivity to reclaim server memory. A scheduled ping keeps the database awake and active.

* **Why this approach, not an alternative:**
  * *Dedicated Table vs Querying User Tables:* Querying private tables like `users` or `skin_assessments` in automated cron jobs risks leaking user data into workflow log files. Using an empty, dedicated `_heartbeat` table isolates keepalive pings safely.

* **Connects to:**
  * **Interacts with:** GitHub Actions automated cron workflow pinging Supabase via the PostgREST API.

---

### `src/pages/Results.jsx`

* **What it does:** Displays the personalized skincare routine generated by AI. It manages recommendation retrieval and caching, triggers the Edge Function if no cached routine exists, renders responsive product cards with ingredients and usage steps, provides direct shopping links, and facilitates favorite saving and feedback collection.

* **Full walkthrough:**
```javascript
import { useEffect, useState } from 'react';
import { useParams, useNavigate, Link, useLocation } from 'react-router-dom';
import { useAuth } from '@/contexts/AuthContext';
import { supabase } from '@/integrations/supabase/client';
import { Product } from '@/types/skincare';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Loader2, Leaf, ExternalLink, AlertTriangle, ArrowLeft, RefreshCw, Heart, History } from 'lucide-react';
import { useToast } from '@/hooks/use-toast';
import { ProductFeedback } from '@/components/ProductFeedback';
import { FavoriteButton } from '@/components/FavoriteButton';
import { BrandLogo } from '@/components/BrandLogo';

export default function Results() {
  const { assessmentId } = useParams();
  const { user } = useAuth();
  const navigate = useNavigate();
  const location = useLocation();
  const { toast } = useToast();
  const facePhoto = (location.state as any)?.facePhoto as string | null;
```
1. Extracts `assessmentId` from URL route params and `facePhoto` from `location.state`.

```javascript
  const fetchRecommendations = async () => {
    if (!assessmentId) return;
    setError(null);

    try {
      // 1. Check if recommendations already exist in Postgres cache
      const { data: existing } = await supabase
        .from('recommendations')
        .select('*')
        .eq('assessment_id', assessmentId)
        .maybeSingle();

      if (existing) {
        setRecommendationId(existing.id);
        setProducts(existing.products as unknown as Product[]);
        setAiSummary(existing.ai_summary || '');
        setHasFetched(true);
        return;
      }

      // 2. Not cached: fetch the assessment profile from database
      setIsLoading(true);
      setIsGenerating(true);

      const { data: assessment, error: assessmentError } = await supabase
        .from('skin_assessments')
        .select('*')
        .eq('id', assessmentId)
        .single();

      if (assessmentError) throw assessmentError;

      // 3. Invoke the Edge Function with assessment and optional photo
      const { data, error: fnError } = await supabase.functions.invoke('get-recommendations', {
        body: { assessment, facePhoto },
      });

      if (fnError) throw fnError;

      setProducts(data.products || []);
      setAiSummary(data.summary || '');

      // 4. Save newly generated recommendations to cache them permanently
      const { data: savedRec, error: saveError } = await supabase.from('recommendations').insert({
        assessment_id: assessmentId,
        user_id: user!.id,
        products: data.products,
        ai_summary: data.summary,
      }).select('id').single();

      if (saveError) throw saveError;
      if (savedRec) setRecommendationId(savedRec.id);

    } catch (err) {
      console.error('Error fetching recommendations:', err);
      setError('Failed to generate recommendations. Please try again.');
      toast({ title: 'Error', description: 'Failed to load recommendations', variant: 'destructive' });
    } finally {
      setIsLoading(false);
      setIsGenerating(false);
      setHasFetched(true);
    }
  };
```
2. **Caching & Execution Flow:**
   * **Cache Lookup:** Checks if `recommendations` already has a row for `assessment_id`. If found, displays cached products immediately, avoiding redundant AI costs when revisiting from History.
   * **Generation Call:** If not cached, fetches the assessment record from `skin_assessments`, then invokes `supabase.functions.invoke('get-recommendations', { body: { assessment, facePhoto } })`.
   * **Persistence:** Persists the returned products and summary into the `recommendations` table so future visits load instantly.

```javascript
        <div className="grid gap-8 md:grid-cols-2">
          {products.map((product, index) => (
            <Card key={index} className="shadow-soft-lg border-border/50 overflow-hidden feature-card rounded-2xl animate-fade-up" style={{ animationDelay: `${index * 100}ms` }}>
              <CardHeader className="pb-3 bg-skinwise-cream/50">
                <div className="flex justify-between items-start">
                  <div>
                    <p className="text-sm text-skinwise-terracotta font-semibold uppercase tracking-wider">{product.brand}</p>
                    <CardTitle className="text-xl font-display mt-1 text-skinwise-bark">{product.name}</CardTitle>
                  </div>
                  <div className="flex items-center gap-2">
                    <Badge variant="secondary" className="bg-background/80 backdrop-blur-sm">~{product.priceRange}</Badge>
                    {user && <FavoriteButton product={product} userId={user.id} />}
                  </div>
                </div>
              </CardHeader>
              <CardContent className="space-y-4 pt-4">
                <p className="text-sm text-muted-foreground font-body leading-relaxed">{product.whySuitable}</p>
                
                {product.usageInstructions && (
                  <div className="bg-skinwise-sage/5 p-3 rounded-lg border border-skinwise-sage/10">
                    <p className="text-xs text-skinwise-bark font-medium mb-1">How to use:</p>
                    <p className="text-xs text-muted-foreground font-body">{product.usageInstructions}</p>
                  </div>
                )}
                
                {product.keyIngredients?.length > 0 && (
                  <div className="flex flex-wrap gap-1.5 pt-2">
                    {product.keyIngredients.map((ing, i) => (
                       <Badge key={i} variant="outline" className="text-[10px] uppercase tracking-wider">{ing}</Badge>
                    ))}
                  </div>
                )}
                
                {product.productUrl && (
                  <Button className="w-full rounded-full shadow-sm mt-2" variant="outline" asChild>
                    <a href={product.productUrl} target="_blank" rel="noopener noreferrer">
                      Shop Product <ExternalLink className="w-3 h-3 ml-2" />
                    </a>
                  </Button>
                )}
                
                {recommendationId && user && (
                  <div className="pt-4 border-t border-border/30">
                    <ProductFeedback
                      recommendationId={recommendationId}
                      productName={product.name}
                      userId={user.id}
                    />
                  </div>
                )}
              </CardContent>
            </Card>
          ))}
        </div>
```
3. **Card UI Architecture:**
   * Displays Brand, Product Name, and Price Range badge.
   * Embeds `<FavoriteButton product={product} userId={user.id} />` to bookmark items.
   * Renders clinical explanations (`whySuitable`), structured instructions (`usageInstructions`), and ingredient badges.
   * Renders external shopping link button (`Shop Product`) opening in a secure new tab (`rel="noopener noreferrer"`).
   * Embeds `<ProductFeedback />` allowing users to rate and review individual products.
   * Includes a medical disclaimer alerting users that results are AI suggestions, not official dermatological diagnoses.

* **Concepts a beginner might not know:**
  * **Database Caching:** Storing expensive computational results (like AI generations) in a database table to serve subsequent requests instantly without paying API costs twice.
  * **Target `_blank` & `rel="noopener noreferrer"`:** Opening an external link in a new browser tab with `target="_blank"` presents a security vulnerability known as "reverse tabnabbing" unless secured with `rel="noopener noreferrer"`, which prevents the target page from accessing `window.opener`.

* **Why this approach, not an alternative:**
  * *`hasFetched` Gate Pattern:* Notice line 29 (`const [hasFetched, setHasFetched] = useState(false)`). In React 18, `useEffect` runs twice on mount in development Strict Mode. Without the `hasFetched` guard, the component would fire two duplicate Edge Function calls and double the Gemini AI API costs!

* **Connects to:**
  * **Imports from:** [AuthContext.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/contexts/AuthContext.jsx), [client.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/integrations/supabase/client.js), [skincare.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/types/skincare.js), [FavoriteButton.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/FavoriteButton.jsx), [ProductFeedback.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/ProductFeedback.jsx), and [BrandLogo.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/BrandLogo.jsx).
  * **Invokes:** Edge Function `get-recommendations`.
  * **Navigates to:** `/quiz`, `/favorites`, and `/history`.

---

### `src/components/BrandLogo.jsx`

* **What it does:** Renders the application's vector identity logo. It combines an SVG canvas with animated pulsing star glyphs, dual Google Web Fonts typography (cursive script "Skin" and serif "wise"), and theme-responsive color variants.

* **Full walkthrough:**
```javascript
import React from 'react';
import { cn } from "@/lib/utils";

interface BrandLogoProps {
  className?: string;
  variant?: 'default' | 'white';
}

export function BrandLogo({ className, variant = 'default' }: BrandLogoProps) {
  const isWhite = variant === 'white';
  
  // Colors sampled from the original design
  const skinColor = isWhite ? "white" : "#c39b6d";
  const wiseColor = isWhite ? "white" : "#f0b24f";
  const starColor = isWhite ? "white" : "#f0b24f";

  return (
    <svg
      viewBox="0 0 160 100"
      fill="none"
      xmlns="http://www.w3.org/2000/svg"
      className={cn("w-auto h-auto", className)}
      style={{ filter: isWhite ? 'none' : 'drop-shadow(0 2px 4px rgba(0,0,0,0.05))' }}
    >
      <defs>
        <path id="tapered-star" d="M10 0C10 0 11.5 7.5 18 10C11.5 12.5 10 20 10 20C10 20 8.5 12.5 2 10C8.5 7.5 10 0 10 0Z" />
      </defs>

      {/* Top Left Small Star */}
      <use 
        href="#tapered-star" 
        x="15" 
        y="15" 
        transform="scale(0.4)" 
        fill={starColor} 
        className="animate-pulse"
        style={{ transformOrigin: '20px 20px', animationDuration: '3s' }}
      />

      {/* Right Large Star */}
      <use 
        href="#tapered-star" 
        x="115" 
        y="35" 
        transform="scale(1.2)" 
        fill={starColor} 
        className="animate-pulse"
        style={{ transformOrigin: '125px 45px', animationDuration: '4s', animationDelay: '0.5s' }}
      />

      {/* Bottom Left Small Star */}
      <use 
        href="#tapered-star" 
        x="20" 
        y="65" 
        transform="scale(0.5)" 
        fill={starColor} 
        className="animate-pulse"
        style={{ transformOrigin: '25px 75px', animationDuration: '2.5s', animationDelay: '1s' }}
      />

      {/* "Skin" - Elegant Brush Script */}
      <text
        x="80"
        y="45"
        textAnchor="middle"
        fill={skinColor}
        style={{ 
          fontFamily: "'Great Vibes', cursive", 
          fontSize: '48px',
          fontWeight: 'normal'
        }}
      >
        Skin
      </text>

      {/* "wise" - Chunky Rounded Serif */}
      <text
        x="88"
        y="75"
        textAnchor="middle"
        fill={wiseColor}
        style={{ 
          fontFamily: "'Bowlby One SC', serif", 
          fontSize: '28px',
          fontWeight: 'normal',
          letterSpacing: '0.05em'
        }}
      >
        wise
      </text>
    </svg>
  );
}
```
1. Defines an SVG canvas with `viewBox="0 0 160 100"`.
2. **SVG Defs & Reusable Symbols:** Declares a reusable 4-pointed sparkle path `<path id="tapered-star">` inside `<defs>`. Instances are cloned across the coordinate plane using `<use href="#tapered-star" ... />` with staggered CSS pulse animation timings (`animationDuration`, `animationDelay`).
3. **Dual Font Typography:** Renders `<text>` nodes styled with Google Web Fonts loaded in `index.css`: `'Great Vibes'` (for cursive "Skin") and `'Bowlby One SC'` (for all-caps "wise").

* **Concepts a beginner might not know:**
  * **SVG `<defs>` and `<use>`:** An SVG optimization pattern where a graphic shape is declared once in memory inside `<defs>` (definitions) and re-used multiple times across the canvas with different coordinates and scales, drastically reducing SVG file size.
  * **SVG `viewBox` Coordinate Space:** An attribute defining the internal aspect ratio and coordinate grid of an SVG, allowing the vector icon to scale infinitely to any screen size without distortion.

* **Why this approach, not an alternative:**
  * *Inline SVG Component vs. Bitmap Image (`.png` / `.jpg`):* A bitmap PNG logo pixelates on high-density 4K Retina screens and requires separate files for dark and light mode. Rendering the logo as a React SVG component ensures sharpness at any resolution and allows programmatic fill changes and CSS micro-animations.

* **Connects to:**
  * **Imported by:** Header navigation in [Index.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Index.jsx), [Auth.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Auth.jsx), [Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx), [Results.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Results.jsx), [History.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/History.jsx), [Favorites.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Favorites.jsx), and [FaceCapturePage.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/FaceCapturePage.jsx).

---

### `src/components/FavoriteButton.jsx`

* **What it does:** An interactive heart icon button rendered on product cards that toggles product bookmarking in the user's Supabase database.

* **Full walkthrough:**
```javascript
import { useState, useEffect } from 'react';
import { Heart } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { supabase } from '@/integrations/supabase/client';
import { useToast } from '@/hooks/use-toast';
import { Product } from '@/types/skincare';
import { cn } from '@/lib/utils';

interface FavoriteButtonProps {
  product: Product;
  userId: string;
}

export function FavoriteButton({ product, userId }: FavoriteButtonProps) {
  const [isFavorite, setIsFavorite] = useState(false);
  const [isLoading, setIsLoading] = useState(false);
  const { toast } = useToast();

  useEffect(() => {
    checkIfFavorite();
  }, [product.name, userId]);

  const checkIfFavorite = async () => {
    const { data } = await supabase
      .from('favorites')
      .select('id')
      .eq('user_id', userId)
      .eq('product_name', product.name)
      .maybeSingle();
    
    setIsFavorite(!!data);
  };
```
1. On mount and whenever `product.name` or `userId` changes, queries the `favorites` table for an existing bookmark matching both `user_id` and `product_name`.

```javascript
  const toggleFavorite = async () => {
    setIsLoading(true);
    try {
      if (isFavorite) {
        // Remove from favorites
        const { error } = await supabase
          .from('favorites')
          .delete()
          .eq('user_id', userId)
          .eq('product_name', product.name);

        if (error) throw error;
        setIsFavorite(false);
        toast({ title: 'Removed from favorites' });
      } else {
        // Add to favorites
        const { error } = await supabase
          .from('favorites')
          .insert({
            user_id: userId,
            product_name: product.name,
            product_brand: product.brand,
            product_data: product as unknown as Record<string, unknown>,
          } as any);

        if (error) throw error;
        setIsFavorite(true);
        toast({ title: 'Added to favorites!' });
      }
    } catch (err) {
      console.error('Error toggling favorite:', err);
      toast({ title: 'Error', description: 'Failed to update favorites', variant: 'destructive' });
    } finally {
      setIsLoading(false);
    }
  };
```
2. **Toggle Operation:**
   * If already favorited: executes a database `DELETE` and sets `isFavorite(false)`.
   * If not favorited: executes an `INSERT` saving `user_id`, `product_name`, `product_brand`, and the full serialized product object in the PostgreSQL JSON column `product_data`.

* **Concepts a beginner might not know:**
  * **PostgreSQL JSON / JSONB Columns:** A column data type that stores raw JSON objects and arrays directly inside a relational PostgreSQL table, allowing flexible schema-less data storage alongside strictly typed relational columns.
  * **`maybeSingle()`:** A Supabase PostgREST query method that returns either 1 matching record or `null` without throwing a 406/PGRST116 runtime error if 0 rows are found (unlike `.single()`, which throws an error if no row exists).

* **Why this approach, not an alternative:**
  * *Storing Snapshot in JSON (`product_data`):* Storing the complete `product` object in a `product_data` JSON column ensures that even if products change in future AI generations, the user's bookmarked version (with its price, ingredients, and shopping URL) remains permanently readable without complex relational foreign key dependencies.

* **Connects to:**
  * **Imports from:** [client.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/integrations/supabase/client.js), [use-toast.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/hooks/use-toast.js), [skincare.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/types/skincare.js), and [utils.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/lib/utils.js).
  * **Used inside:** [Results.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Results.jsx).

---

### `src/components/ProductFeedback.jsx`

* **What it does:** Renders a product review dialog component. Displays a 5-star rating interface with hover effects, an "I've tried this product" checkbox, and a 500-character feedback textarea that upserts records into Supabase.

* **Full walkthrough:**
```javascript
import { useState, useEffect } from 'react';
import { Star, MessageSquare, Check } from 'lucide-react';
import { Button } from '@/components/ui/button';
import { Textarea } from '@/components/ui/textarea';
import { Checkbox } from '@/components/ui/checkbox';
import { Label } from '@/components/ui/label';
import {
  Dialog,
  DialogContent,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from '@/components/ui/dialog';
import { supabase } from '@/integrations/supabase/client';
import { useToast } from '@/hooks/use-toast';
import { cn } from '@/lib/utils';

interface ProductFeedbackProps {
  recommendationId: string;
  productName: string;
  userId: string;
}
```
1. Defines props linking feedback to the parent `recommendationId`, `productName`, and `userId`.

```javascript
  const fetchExistingFeedback = async () => {
    const { data } = await supabase
      .from('product_feedback')
      .select('rating, feedback, tried_product')
      .eq('recommendation_id', recommendationId)
      .eq('product_name', productName)
      .eq('user_id', userId)
      .maybeSingle();

    if (data) {
      setExistingFeedback(data);
      setRating(data.rating);
      setFeedback(data.feedback || '');
      setTriedProduct(data.tried_product || false);
    }
  };

  const handleSubmit = async () => {
    if (rating === 0) {
      toast({ title: 'Please select a rating', variant: 'destructive' });
      return;
    }

    setIsLoading(true);
    try {
      if (existingFeedback) {
        const { error } = await supabase
          .from('product_feedback')
          .update({
            rating,
            feedback: feedback.trim() || null,
            tried_product: triedProduct,
          })
          .eq('recommendation_id', recommendationId)
          .eq('product_name', productName)
          .eq('user_id', userId);
        if (error) throw error;
      } else {
        const { error } = await supabase.from('product_feedback').insert({
          user_id: userId,
          recommendation_id: recommendationId,
          product_name: productName,
          rating,
          feedback: feedback.trim() || null,
          tried_product: triedProduct,
        });
        if (error) throw error;
      }

      setExistingFeedback({ rating, feedback, tried_product: triedProduct });
      toast({ title: 'Feedback saved!' });
      setOpen(false);
    } catch (error) { ... }
  };
```
2. **Review Persistence:** Loads existing feedback on mount. On submit, differentiates between updating an existing review via `.update()` and recording a new one via `.insert()`.
3. **Interactive Star Rating:** Tracks `hoverRating` and `rating` state. Hovering over the 4th star temporarily illuminates stars 1–4; clicking commits the rating value permanently.

* **Concepts a beginner might not know:**
  * **Radix UI Dialog (Modal):** An accessible overlay window that intercepts keyboard focus, traps focus inside the modal dialog while open, and closes gracefully on `Escape` key press.
  * **Hover State Tracking:** Maintaining temporary state in React (`onMouseEnter` and `onMouseLeave`) to preview user choices before a click event commits them.

* **Why this approach, not an alternative:**
  * *Modal Dialog vs In-Card Form:* Displaying a rating and comment form inline on every product card would clutter the results view. Packaging it in a headless `<Dialog>` keeps the product catalog clean.

* **Connects to:**
  * **Imports from:** [client.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/integrations/supabase/client.js), [use-toast.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/hooks/use-toast.js), [utils.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/lib/utils.js), and shadcn UI Dialog components.
  * **Used inside:** [Results.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Results.jsx).

---

### `src/components/ThemeToggle.jsx`

* **What it does:** Renders the dark/light mode toggle button in the header bar with smooth rotating icon transitions.

* **Full walkthrough:**
```javascript
import { Moon, Sun } from "lucide-react";
import { useTheme } from "next-themes";
import { Button } from "@/components/ui/button";

export function ThemeToggle() {
  const { setTheme, theme } = useTheme();

  const toggleTheme = () => {
    setTheme(theme === "dark" ? "light" : "dark");
  };

  return (
    <Button
      variant="ghost"
      size="icon"
      className="h-9 w-9 rounded-full hover:bg-skinwise-terracotta/10 transition-colors hover:text-inherit"
      onClick={toggleTheme}
    >
      <Sun className="h-4 w-4 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0 text-skinwise-terracotta" />
      <Moon className="absolute h-4 w-4 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100 text-skinwise-sage" />
      <span className="sr-only">Toggle theme</span>
    </Button>
  );
}
```
1. Uses `useTheme()` from `next-themes` to inspect and toggle the active theme between `"dark"` and `"light"`.
2. **Rotating CSS Transitions:**
   * In light mode: `<Sun />` has `rotate-0 scale-100` (visible), while `<Moon />` has `rotate-90 scale-0` (hidden).
   * In dark mode: Tailwind's `dark:` modifier inverts them: `<Sun />` spins out with `dark:-rotate-90 dark:scale-0`, while `<Moon />` rotates in with `dark:rotate-0 dark:scale-100`.

* **Concepts a beginner might not know:**
  * **`next-themes`:** An abstraction that persists theme preference in `localStorage`, syncs with the user's operating system setting (`prefers-color-scheme`), and prevents dark-mode page flickering on initial load.

* **Why this approach, not an alternative:**
  * *Tailwind `class` Strategy:* Toggling a `.dark` CSS class on the root `<html>` element is significantly faster than storing separate CSS stylesheets and re-downloading them.

* **Connects to:**
  * **Imports from:** `next-themes`, `lucide-react`, and [button.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/ui/button.jsx).
  * **Used inside:** [Index.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Index.jsx).

---

### `src/pages/Favorites.jsx`

* **What it does:** Renders the user's saved favorites dashboard. Displays bookmarked products from PostgreSQL, enables direct deletion, and provides links to external e-commerce sites.

* **Full walkthrough:**
```javascript
import { useEffect, useState } from 'react';
import { Link, useNavigate } from 'react-router-dom';
import { useAuth } from '@/contexts/AuthContext';
import { supabase } from '@/integrations/supabase/client';
import { Product } from '@/types/skincare';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';
import { Badge } from '@/components/ui/badge';
import { Heart, ExternalLink, ArrowLeft, Trash2 } from 'lucide-react';
import { useToast } from '@/hooks/use-toast';
import { BrandLogo } from '@/components/BrandLogo';

interface FavoriteItem {
  id: string;
  product_name: string;
  product_brand: string | null;
  product_data: Product;
  created_at: string;
}
```
1. Authenticates on mount: redirects to `/auth` if the user is unauthenticated.

```javascript
  const fetchFavorites = async () => {
    try {
      const { data, error } = await supabase
        .from('favorites')
        .select('*')
        .eq('user_id', user!.id)
        .order('created_at', { ascending: false });

      if (error) throw error;
      setFavorites((data || []).map(item => ({
        ...item,
        product_data: item.product_data as unknown as Product,
      })));
    } catch (err) {
      console.error('Error fetching favorites:', err);
      toast({ title: 'Error', description: 'Failed to load favorites', variant: 'destructive' });
    } finally {
      setIsLoading(false);
    }
  };

  const removeFavorite = async (id: string) => {
    try {
      const { error } = await supabase
        .from('favorites')
        .delete()
        .eq('id', id);

      if (error) throw error;
      setFavorites(favorites.filter(f => f.id !== id));
      toast({ title: 'Removed from favorites' });
    } catch (err) {
      console.error('Error removing favorite:', err);
      toast({ title: 'Error', description: 'Failed to remove', variant: 'destructive' });
    }
  };
```
2. Queries the `favorites` table ordered by `created_at DESC`.
3. `removeFavorite()` executes an HTTP DELETE through PostgREST and filters the local React state array immediately (`favorites.filter(f => f.id !== id)`).

* **Concepts a beginner might not know:**
  * **Optimistic Local Filtering:** Updating the UI array state immediately after an API call succeeds without executing a costly second `fetchFavorites()` query over the network.

* **Why this approach, not an alternative:**
  * *Separate Favorites Table vs Storing IDs in a User Profile Array:* Storing favorites in a dedicated relational table scales indefinitely, allows easy querying with pagination, and enables individual row deletion with RLS protection.

* **Connects to:**
  * **Imports from:** [AuthContext.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/contexts/AuthContext.jsx), [client.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/integrations/supabase/client.js), and [BrandLogo.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/BrandLogo.jsx).
  * **Referenced by:** [App.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/App.jsx) (`/favorites`).

---

### `src/pages/History.jsx`

* **What it does:** Renders the user's historical timeline of completed skin assessments and AI routines. It includes a sub-query join to fetch recommendations, an expandable accordion view, and an **8-second undo deletion countdown** with optimistic UI removal and database commit mechanisms.

* **Full walkthrough:**
```javascript
import { useState, useEffect, useRef, useCallback } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '@/contexts/AuthContext';
import { supabase } from '@/integrations/supabase/client';
import { Button } from '@/components/ui/button';
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Badge } from '@/components/ui/badge';
import { Skeleton } from '@/components/ui/skeleton';
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from '@/components/ui/alert-dialog';
import { useToast } from '@/hooks/use-toast';
import { ArrowLeft, Calendar, ChevronRight, Sparkles, Droplets, Sun, Trash2 } from 'lucide-react';
import { BrandLogo } from '@/components/BrandLogo';
import { format } from 'date-fns';
import { Product } from '@/types/skincare';

const UNDO_DELAY = 8;

function Countdown({ seconds }: { seconds: number }) {
  const [remaining, setRemaining] = useState(seconds);
  useEffect(() => {
    if (remaining <= 0) return;
    const id = setInterval(() => setRemaining((r) => Math.max(0, r - 1)), 1000);
    return () => clearInterval(id);
  }, [remaining]);
  return <>{remaining}s</>;
}
```
1. **Timer Component (`Countdown`):** Displays a live decrementing seconds indicator inside the undo toast notification.

```javascript
  const fetchHistory = async () => {
    try {
      const { data: assessmentsData, error: assessmentsError } = await supabase
        .from('skin_assessments')
        .select('*')
        .eq('user_id', user!.id)
        .order('created_at', { ascending: false });

      if (assessmentsError) throw assessmentsError;

      const assessmentsWithRecs: AssessmentWithRecommendation[] = [];
      
      for (const assessment of assessmentsData || []) {
        const { data: recData } = await supabase
          .from('recommendations')
          .select('id, ai_summary, products')
          .eq('assessment_id', assessment.id)
          .maybeSingle();

        assessmentsWithRecs.push({
          ...assessment,
          recommendation: recData ? {
            ...recData,
            products: recData.products as unknown as Product[]
          } : undefined
        });
      }

      setAssessments(assessmentsWithRecs);
    } catch (error) { ... }
  };
```
2. **Relational Assembly:** Fetches all user assessments, loops through each assessment, and pairs it with its associated AI routine from the `recommendations` table.

```javascript
  const commitDelete = useCallback(async (items: AssessmentWithRecommendation[]) => {
    if (!items.length) return;
    try {
      for (const item of items) {
        await supabase.from('recommendations').delete().eq('assessment_id', item.id);
        await supabase.from('skin_assessments').delete().eq('id', item.id);
      }
      isCommittedRef.current = true;
    } catch (err) {
      console.error('Delete commit exception:', err);
    }
  }, []);

  const undoPendingDelete = useCallback(async () => {
    const pending = pendingDeleteRef.current;
    if (!pending) return;

    if (pendingDeleteTimer.current) {
      clearTimeout(pendingDeleteTimer.current);
      pendingDeleteTimer.current = null;
    }

    setAssessments(prev => {
      const restored = [...prev, ...pending.items];
      return restored.sort((a, b) => new Date(b.created_at).getTime() - new Date(a.created_at).getTime());
    });

    pendingDeleteRef.current = null;
    setPendingDelete(null);
    toast({ title: 'Deletion cancelled', description: 'Your assessment was restored.' });
  }, []);
```
3. **8-Second Undo Architecture:**
   * When the user clicks Delete, items are *immediately removed from local state* (`optimistic update`), making the interface feel instantaneous.
   * A reference to deleted items is stored in `pendingDeleteRef.current`.
   * A timer starts for 8 seconds (`UNDO_DELAY = 8`). A toast appears with an `<Undo>` action and live countdown.
   * If the user clicks Undo before 8 seconds elapse, `clearTimeout()` cancels the deletion and restores items to state sorted chronologically.
   * If the 8 seconds expire, `commitDelete()` executes the real database deletion against Supabase.
   * If the user closes or leaves the page before the timer expires, the unmount cleanup hook flushes `commitDelete()` immediately, ensuring database integrity.

* **Concepts a beginner might not know:**
  * **Optimistic UI:** A frontend pattern where the UI reflects the result of an action before the backend confirms it, giving the impression of zero network latency.
  * **React `useRef` as Mutable Storage:** A hook that holds a mutable value that persists across renders without triggering a component re-render when changed (used here to hold timers and uncommitted items).
  * **Unmount Cleanup Flush:** Running pending asynchronous side effects in `useEffect`'s return cleanup function before the component is destroyed.

* **Why this approach, not an alternative:**
  * *Undo Pattern vs Immediate Modal Deletion:* Accidental clicks on mobile touchscreens are common. Providing an 8-second grace period with optimistic hiding gives users a safety net while preserving instant interface responsiveness.

* **Connects to:**
  * **Imports from:** [AuthContext.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/contexts/AuthContext.jsx), [client.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/integrations/supabase/client.js), `date-fns`, and shadcn AlertDialog UI components.
  * **Referenced by:** [App.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/App.jsx) (`/history`).

---

### `src/pages/Profile.jsx`

* **What it does:** Allows users to view their account email and update their full name, persisting profile metadata into the `profiles` table.

* **Full walkthrough:**
```javascript
import { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import { useAuth } from '@/contexts/AuthContext';
import { supabase } from '@/integrations/supabase/client';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { Label } from '@/components/ui/label';
import { Card, CardContent, CardDescription, CardHeader, CardTitle } from '@/components/ui/card';
import { Skeleton } from '@/components/ui/skeleton';
import { useToast } from '@/hooks/use-toast';
import { ArrowLeft, User, Mail, Save, Loader2 } from 'lucide-react';
import { z } from 'zod';

const profileSchema = z.object({
  fullName: z.string().trim().max(100, 'Name must be less than 100 characters').optional(),
});
```
1. Declares a Zod schema validating that the name is trimmed and does not exceed 100 characters.

```javascript
  const fetchProfile = async () => {
    try {
      const { data, error } = await supabase
        .from('profiles')
        .select('full_name, email')
        .eq('user_id', user!.id)
        .maybeSingle();

      if (error) throw error;
      if (data) {
        setFullName(data.full_name || '');
        setEmail(data.email || user!.email || '');
      } else {
        setEmail(user!.email || '');
      }
    } catch (error) { ... }
  };

  const handleSave = async () => {
    setError('');
    const result = profileSchema.safeParse({ fullName });
    if (!result.success) {
      setError(result.error.errors[0].message);
      return;
    }

    setIsSaving(true);
    try {
      const { error } = await supabase
        .from('profiles')
        .upsert({
          user_id: user!.id,
          email: user!.email || null,
          full_name: fullName.trim() || null,
          updated_at: new Date().toISOString(),
        }, { onConflict: 'user_id' });

      if (error) throw error;
      toast({ title: 'Profile Updated' });
    } catch (error) { ... }
  };
```
2. Loads profile from `profiles` on mount.
3. **Database Upsert:** Calls `supabase.from('profiles').upsert(..., { onConflict: 'user_id' })`. If a profile row already exists for this `user_id`, Postgres updates it; if not, Postgres creates it.

* **Concepts a beginner might not know:**
  * **Database Upsert (Insert or Update):** An atomic database operation that inserts a new row if it doesn't exist, or updates the existing row if a duplicate primary/unique key constraint is encountered.

* **Why this approach, not an alternative:**
  * *`upsert` vs `insert` with error catching:* Calling `upsert` eliminates race conditions where two simultaneous updates try to create duplicate profile rows.

* **Connects to:**
  * **Imports from:** [AuthContext.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/contexts/AuthContext.jsx), [client.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/integrations/supabase/client.js), and [use-toast.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/hooks/use-toast.js).
  * **Referenced by:** [App.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/App.jsx) (`/profile`).

---

### `src/pages/Index.jsx`

* **What it does:** The main public landing page. Features the brand hero section, value propositions, navigation header with authentication-aware links, and call-to-action buttons.

* **Full walkthrough:**
```javascript
import { useNavigate } from 'react-router-dom';
import { useAuth } from '@/contexts/AuthContext';
import { Button } from '@/components/ui/button';
import { ThemeToggle } from '@/components/ThemeToggle';
import { ChevronRight, Heart, History, LogOut, User, Zap } from 'lucide-react';
import { BrandLogo } from '@/components/BrandLogo';

export default function Index() {
  const navigate = useNavigate();
  const { user, signOut, loading } = useAuth();
```
1. **Dynamic Authentication State:**
   * If user is signed in: Renders navigation buttons for History, Favorites, Profile, and Sign Out. The hero CTA reads `"Take the Skin Survey"`, linking directly to `/quiz`.
   * If user is a visitor: Renders a `"Sign In"` button and `"Get Started"` CTA that routes to `/auth`.
2. **Earthy Spa Typography & Styling:**
   * Configured with custom CSS clamp values: `font-size: clamp(3rem, 7vw, 5.5rem)` ensuring responsive typography scaling from mobile phones to ultra-wide displays.
   * Utilizes warm palette color tokens: `--skinwise-terracotta`, `--skinwise-sage`, and `--skinwise-bark`.

* **Concepts a beginner might not know:**
  * **CSS `clamp(min, preferred, max)`:** A modern CSS mathematical function that defines a fluid value that scales dynamically with viewport width while respecting strict minimum and maximum boundaries.

* **Why this approach, not an alternative:**
  * *State-Aware Landing Page:* Seamlessly serving both unauthenticated marketing visitors and returning logged-in users from the same root URL (`/`) simplifies routing and improves user retention.

* **Connects to:**
  * **Imports from:** [AuthContext.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/contexts/AuthContext.jsx), [BrandLogo.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/BrandLogo.jsx), [ThemeToggle.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/ThemeToggle.jsx), and UI buttons.
  * **Routes to:** `/quiz`, `/auth`, `/history`, `/favorites`, and `/profile`.

---

### `src/pages/NotFound.jsx`

* **What it does:** The catch-all 404 page for unmatched routes. Logs missing URLs to the developer console and offers a return link to the homepage.

* **Full walkthrough:**
```javascript
import { useLocation } from "react-router-dom";
import { useEffect } from "react";

const NotFound = () => {
  const location = useLocation();

  useEffect(() => {
    console.error("404 Error: User attempted to access non-existent route:", location.pathname);
  }, [location.pathname]);

  return (
    <div className="flex min-h-screen items-center justify-center bg-muted">
      <div className="text-center">
        <h1 className="mb-4 text-4xl font-bold">404</h1>
        <p className="mb-4 text-xl text-muted-foreground">Oops! Page not found</p>
        <a href="/" className="text-primary underline hover:text-primary/90">
          Return to Home
        </a>
      </div>
    </div>
  );
};

export default NotFound;
```
1. Reads `location.pathname` via `useLocation()`.
2. Emits an error in browser developer tools to aid debugging broken links.
3. Renders a centered 404 message with a home link.

* **Concepts a beginner might not know:**
  * **Wildcard Route (`*`):** A router path pattern that matches any URL that did not match any of the preceding routes in the route table.

* **Why this approach, not an alternative:**
  * *Critique / Improvement Point:* Notice line 16 uses standard anchor `<a href="/">` instead of React Router's `<Link to="/">`. A standard anchor causes a full browser document reload, whereas `<Link>` would perform client-side transition without re-downloading JavaScript bundles. In a production update, replacing `<a href="/">` with `<Link to="/">` is recommended.

* **Connects to:**
  * **Bound to:** `<Route path="*" element={<NotFound />} />` in [App.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/App.jsx).

---

## 4. End-to-End Flow Traces

To truly understand how SkinWise functions as a complete distributed software system, we must trace how data, network packets, and application state move across components during real user interactions. Below are the 4 foundational user action flows, traced step-by-step in true execution order from the user interface down to external AI endpoints, database tables, and back.

```
       ===========================================================
                     SKINWISE END-TO-END FLOW OVERVIEW
       ===========================================================
 
 [Flow 1: User Onboarding & Auth]
   User Action (Auth.jsx) ---> Zod Validation ---> GoTrue API ---> JWT in LocalStorage ---> ProtectedRoute Passes
 
 [Flow 2: Assessment & Computer Vision]
   Wizard Steps 1-5 ---> Step 6 Camera Mounts ---> getUserMedia() Stream
                                                         │
                                        ┌────────────────┴───────────────┐
                                        ▼                                ▼
                              Laplacian Variance (Blur)         Rec.709 Luminance
                                        │                                │
                                        └────────────────┬───────────────┘
                                                         │
                                               BlazeFace WebGL Inference
                                                         │
                                                         ▼
                                            Passed Oval Threshold & Scale
                                                         │
                                                         ▼
                                           Insert to 'skin_assessments' (RLS)
 
 [Flow 3: AI Recommendation & E-Commerce Scraping]
   Results.jsx Mounts ---> Edge Function Invocation ('get-recommendations')
                                 │
                 ┌───────────────┴───────────────┐
                 ▼                               ▼
       Gemini 2.5 Flash API              Firecrawl Search Scrape
      (Multimodal Diagnosis)             (Indian E-Commerce Links)
                 │                               │
                 └───────────────┬───────────────┘
                                 ▼
                     Upsert to 'recommendations'
                                 │
                                 ▼
               Render Tabbed Routine & Product Cards
 
 [Flow 4: History Management & Safe Undo Deletion]
   History.jsx Mounts ---> Relational Query (skin_assessments + recommendations)
                                 │
                                 ▼
                        User clicks "Delete"
                                 │
                                 ▼
                    Optimistic Local State Eviction
                                 │
                                 ▼
                      8-Second Toast Countdown
                     ┌───────────┴───────────┐
                     ▼                       ▼
            [User Clicks Undo]     [Timer Reaches 0s]
                     │                       │
           Item Restored to State     Postgres Cascade DELETE
           (Zero DB mutations)        (Row permanently evicted)
```

---

### Flow 1: User Onboarding and Authentication Flow

This flow traces what happens when a new user visits SkinWise, registers with an email and password, receives their authentication session, and is granted access to protected application routes.

#### Step-by-Step Execution Sequence

1. **User Interaction on `/auth` ([Auth.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Auth.jsx)):**
   * The user fills in the `email` and `password` input fields on the login/signup form.
   * The user clicks the **"Create Account"** button.
   * The `<form onSubmit={handleSubmit}>` event triggers, executing `e.preventDefault()` to stop the browser's native full-page document reload.

2. **Client-Side Form Validation ([Auth.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Auth.jsx)):**
   * The form submission handler validates inputs using the pre-compiled Zod schema:
     ```javascript
     const authSchema = z.object({
       email: z.string().trim().email("Please enter a valid email address"),
       password: z.string().min(6, "Password must be at least 6 characters")
     });
     ```
   * If validation fails (e.g., password is 4 characters), execution halts immediately, local validation state updates, and an inline error is displayed without making any network request.
   * If validation succeeds, `setIsLoading(true)` renders a spinner on the submit button.

3. **Delegation to Authentication Context ([AuthContext.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/contexts/AuthContext.jsx)):**
   * `Auth.jsx` invokes the context method:
     ```javascript
     const { signUp } = useAuth();
     await signUp(email, password);
     ```
   * `signUp` forwards the credentials to the Supabase client:
     ```javascript
     const { data, error } = await supabase.auth.signUp({ email, password });
     ```

4. **Network Request to Supabase GoTrue Auth Service ([client.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/integrations/supabase/client.js)):**
   * The Supabase JavaScript client issues an HTTPS POST request to:
     `https://<project-ref>.supabase.co/auth/v1/signup`
   * Headers include `apikey: VITE_SUPABASE_PUBLISHABLE_KEY` and `Content-Type: application/json`.
   * The payload contains `{ "email": "...", "password": "..." }`.

5. **Server-Side Authentication & Database Record Creation:**
   * Supabase's GoTrue backend receives the request.
   * It hashes the password using bcrypt with a cryptographic salt.
   * It creates a record in the internal `auth.users` system table.
   * If email confirmation is disabled (or once confirmed), GoTrue issues two tokens:
     * **Access Token:** A short-lived JSON Web Token (JWT) containing the user's `sub` (UUID) and role (`authenticated`), signed with Supabase's JWT secret.
     * **Refresh Token:** A long-lived random token used to acquire new access tokens when the JWT expires.

6. **Local Token Persistence & State Broadcasting ([AuthContext.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/contexts/AuthContext.jsx)):**
   * The Supabase client automatically stores the access token and refresh token in browser `localStorage` under the key `sb-<project-ref>-auth-token`.
   * Supabase's internal auth state machine triggers its listener:
     ```javascript
     supabase.auth.onAuthStateChange((_event, session) => {
       setSession(session);
       setUser(session?.user ?? null);
       setLoading(false);
     });
     ```
   * The `AuthProvider` updates its React state (`user: User, session: Session, loading: false`).
   * Because `AuthContext` wraps the entire application tree, this state change triggers a reactive re-render across all consumer components.

7. **Route Authorization Evaluation ([ProtectedRoute.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/ProtectedRoute.jsx)):**
   * When the user attempts to access `/quiz`, `/history`, `/favorites`, or `/profile`, `ProtectedRoute` reads `useAuth()`:
     ```javascript
     if (loading) return <LoadingSpinner />;
     if (!user) return <Navigate to="/auth" state={{ from: location }} replace />;
     return <>{children}</>;
     ```
   * Because `user` is now populated, `ProtectedRoute` unmounts the loading spinner and renders the requested child component.

---

### Flow 2: Assessment Wizard, Computer Vision Quality Gating, and Photo Capture

This flow traces how a user answers the clinical skincare assessment questions, opens their web camera, undergoes real-time computer vision quality validation (blur, lighting, and face positioning), captures their facial image, and commits the assessment to the database.

#### Step-by-Step Execution Sequence

1. **Wizard State Initialization ([Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx) & [useAssessment.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/hooks/useAssessment.js)):**
   * User navigates to `/quiz`.
   * `Quiz.jsx` mounts and initializes state via `useAssessment()`:
     ```javascript
     const { state, updateStep, nextStep, prevStep, submitAssessment } = useAssessment();
     ```
   * State begins at `currentStep = 1` with empty values for demographics, skin type, concerns, and lifestyle.
   * `Quiz.jsx` renders `<ProgressIndicator currentStep={1} totalSteps={6} />` alongside the active step component `<StepBasicInfo />`.

2. **Step-by-Step Progression (Steps 1 through 5):**
   * **Step 1 ([StepBasicInfo.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/quiz/StepBasicInfo.jsx)):** User inputs age group and gender identity. `nextStep()` validates inputs and increments `currentStep` to 2.
   * **Step 2 ([StepSkinType.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/quiz/StepSkinType.jsx)):** User selects primary skin type (Oily, Dry, Combination, Sensitive, Normal).
   * **Step 3 ([StepConcerns.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/quiz/StepConcerns.jsx)):** User selects up to 3 concerns (Acne, Hyperpigmentation, Fine Lines, Dullness, Redness).
   * **Step 4 ([StepLifestyle.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/quiz/StepLifestyle.jsx)):** User specifies daily sun exposure, stress level, sleep hours, water intake, and smoking status.
   * **Step 5 ([StepPreferences.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/quiz/StepPreferences.jsx)):** User sets budget range, fragrance preference, vegan requirement, and cruelty-free requirement.
   * Each step executes `updateStep(stepKey, data)` which dispatches an action to the `useAssessment` reducer, updating the immutable state tree.

3. **Camera Stream Acquisition ([FaceCapture.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/FaceCapture.jsx)):**
   * Step 6 renders `<StepFacePhoto />`, which embeds `<FaceCapture onCapture={handleCapture} />`.
   * In `FaceCapture.jsx`, `useEffect` calls `navigator.mediaDevices.getUserMedia({ video: { facingMode: "user", width: { ideal: 1280 }, height: { ideal: 720 } } })`.
   * The browser prompts the user for camera hardware permission.
   * When granted, the resulting `MediaStream` object is assigned to the rendered `<video ref={videoRef} autoPlay playsInline />` element via `videoRef.current.srcObject = stream`.
   * Video stream dimensions (`videoWidth`, `videoHeight`) are resolved when the `onloadedmetadata` event fires.

4. **BlazeFace Model Initialization ([useFaceDetection.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/hooks/useFaceDetection.js)):**
   * The custom hook loads `@tensorflow-models/blazeface` asynchronously:
     ```javascript
     const model = await blazeface.load();
     ```
   * TensorFlow.js configures the WebGL rendering context on the user's GPU to execute neural network matrix multiplications at hardware-accelerated speeds.

5. **Real-Time Computer Vision Frame Analysis Loop ([useFaceDetection.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/hooks/useFaceDetection.js)):**
   * A continuous processing loop runs using `requestAnimationFrame(detectFrame)`:
     * **Step 5a: Frame Drawing:** The current video frame is rendered into a hidden off-screen HTML5 `<canvas>` element matching the video dimensions.
     * **Step 5b: Pixel Data Extraction:** `ctx.getImageData(0, 0, width, height)` extracts a flat `Uint8ClampedArray` containing the RGBA values of every pixel.
     * **Step 5c: ITU-R BT.709 Luminance Gating:**
       `computeLuminance()` samples pixels using human photometric perception weights:
       $$Y = 0.2126R + 0.7152G + 0.0722B$$
       If average luminance is below 40, quality status registers `low_light`. If above 220, it registers `too_bright`.
     * **Step 5d: Laplacian Variance Blur Gating:**
       `computeLaplacianVariance()` convolves a 3x3 second-order derivative kernel:
       $$\begin{bmatrix} 0 & 1 & 0 \\ 1 & -4 & 1 \\ 0 & 1 & 0 \end{bmatrix}$$
       across grayscale luminance pixels to measure high-frequency edge transitions.
       It computes the statistical variance:
       $$\sigma^2 = \frac{1}{N} \sum_{i=1}^N (I_i - \mu)^2$$
       If $\sigma^2 < 100$, the image lacks sharp high-frequency edges and is classified as `blurry`.
     * **Step 5e: Face Boundary & Landmark Gating:**
       `model.estimateFaces(video, false)` outputs bounding box `[topLeft, bottomRight]` and facial landmarks (eyes, nose, mouth, ears).
       The hook calculates:
       * Face size ratio relative to total frame height (must be between 25% and 65%).
       * Center coordinate alignment relative to the SVG oval target zone drawn on the screen.
     * **Step 5f: UI State Synchronization:**
       If all 3 tests pass (`lighting === 'good'`, `isBlurry === false`, `isCentered === true`), the hook sets `isGoodQuality = true`.
       The SVG oval border transitions from amber/red to emerald green, and the capture button is unlocked.

6. **Photo Capture and Data URL Serialization ([FaceCapture.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/components/FaceCapture.jsx)):**
   * The user clicks **"Take Photo"**.
   * `canvas.toDataURL("image/jpeg", 0.85)` extracts the image as a Base64-encoded Data URL string.
   * Camera hardware tracks are immediately shut down via `stream.getTracks().forEach(t => t.stop())` to turn off the physical camera LED indicator.
   * The base64 string is passed up to `Quiz.jsx` via callback.

7. **Consent Verification and Database Persistence ([Quiz.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Quiz.jsx)):**
   * The user reviews the captured photo preview and checks the mandatory medical consent checkbox: *"I understand this is an AI tool and not medical advice."*
   * The user clicks **"Complete Assessment & Analyze"**.
   * `Quiz.jsx` invokes Supabase database insert:
     ```javascript
     const { data, error } = await supabase
       .from("skin_assessments")
       .insert({
         user_id: user.id,
         skin_type: state.skinType,
         concerns: state.concerns,
         age_range: state.basicInfo.ageGroup,
         gender: state.basicInfo.gender,
         sun_exposure: state.lifestyle.sunExposure,
         sleep_hours: state.lifestyle.sleepHours,
         water_intake: state.lifestyle.waterIntake,
         stress_level: state.lifestyle.stressLevel,
         budget_preference: state.preferences.budget,
         fragrance_free: state.preferences.fragranceFree,
         cruelty_free: state.preferences.crueltyFree,
         vegan: state.preferences.vegan,
         image_data_url: photoDataUrl
       })
       .select()
       .single();
     ```
   * PostgreSQL enforces Row-Level Security: the rule `auth.uid() = user_id` succeeds because the user is authenticated.
   * The database assigns a UUID `id` and records `created_at = now()`.

8. **Navigation to Analysis Screen:**
   * Upon successful database return, `Quiz.jsx` executes:
     ```javascript
     navigate(`/results/${data.id}`, {
       state: { assessmentData: data, photoDataUrl }
     });
     ```
   * The user is transitioned to the results dashboard with zero delay.

---

### Flow 3: AI-Driven Multimodal Analysis & Real-Time E-Commerce Scraping Pipeline

This flow traces the most technically complex path in the application: loading the assessment, verifying whether recommendations are already cached in PostgreSQL, invoking the Supabase Edge Function, executing multimodal reasoning via Google Gemini 2.5 Flash, searching live Indian e-commerce sites via Firecrawl, caching the results, and rendering the personalized skincare regimen.

#### Step-by-Step Execution Sequence

1. **Results Page Mounting & Cache Inspection ([Results.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Results.jsx)):**
   * `Results.jsx` mounts with `assessmentId` extracted from the URL route parameter (`useParams()`).
   * It checks if recommendations were already generated for this assessment:
     ```javascript
     const { data: cached } = await supabase
       .from("recommendations")
       .select("*")
       .eq("assessment_id", assessmentId)
       .maybeSingle();
     ```
   * **Cache Hit Scenario:** If `cached` exists, `setRecommendations(cached.recommendation_data)` updates state immediately, and the page renders in milliseconds without contacting external AI APIs.
   * **Cache Miss Scenario:** If no cached record exists, `generateRecommendations()` is invoked.

2. **Supabase Edge Function Invocation ([Results.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Results.jsx)):**
   * `Results.jsx` calls the Edge Function:
     ```javascript
     const { data, error } = await supabase.functions.invoke("get-recommendations", {
       body: {
         assessmentId,
         assessmentData,
         imageDataUrl: photoDataUrl
       }
     });
     ```
   * The Supabase client automatically injects the active user's JWT in the `Authorization: Bearer <token>` header.

3. **Edge Function Gatekeeping & Authorization ([index.js](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/supabase/functions/get-recommendations/index.js)):**
   * Deno Edge Function receives the HTTP request at the global edge network.
   * **CORS Preflight:** If the method is `OPTIONS`, it immediately responds with HTTP 204 and CORS headers (`Access-Control-Allow-Origin: *`, `Access-Control-Allow-Headers: ...`).
   * **Authorization Verification:** Extracts the `Authorization` header, verifies the JWT signature, and retrieves the authenticated user's ID. Requests without a valid token are rejected with HTTP 401.

4. **Multimodal Payload Assembly for Gemini 2.5 Flash:**
   * The Edge Function formats the user's clinical profile (skin type, lifestyle, environmental factors, sensitivities) into a strict dermatological prompt.
   * It strips the `data:image/jpeg;base64,` prefix from the photo to isolate the raw Base64 bytes.
   * It structures the Gemini API request body:
     ```json
     {
       "contents": [
         {
           "parts": [
             { "text": "You are a professional clinical dermatologist... output pure JSON matching this exact schema..." },
             {
               "inline_data": {
                 "mime_type": "image/jpeg",
                 "data": "<raw-base64-bytes>"
               }
             }
           ]
         }
       ],
       "generationConfig": {
         "temperature": 0.2,
         "response_mime_type": "application/json"
       }
     }
     ```

5. **Gemini 2.5 Flash Multimodal Inference Execution:**
   * The Edge Function issues an HTTPS POST request to Google's Generative Language API:
     `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent?key=GEMINI_API_KEY`
   * Gemini analyzes both the visual pixels (erythema/redness, sebum distribution, comedones, fine lines) and the structured text answers (sun exposure, hydration, stress).
   * Gemini generates a comprehensive JSON response containing:
     * `overall_score`: 0–100 skin health metric.
     * `identified_concerns`: Array of detected issues with severity indicators.
     * `analysis_summary`: 2–3 paragraph plain-language clinical diagnosis.
     * `morning_routine`: Step-by-step regimen (Cleanser, Toner, Serum, Moisturizer, Sunscreen).
     * `evening_routine`: Step-by-step night regimen (Double cleanse, Active treatment, Barrier cream).
     * `recommended_products`: List of 4–6 specific product recommendations featuring target ingredients (e.g., *Niacinamide 10% + Zinc 1%*, *Salicylic Acid 2%*).

6. **E-Commerce Real-Time Search & Verification via Firecrawl:**
   * For each recommended product in the routine, the Edge Function queries the Firecrawl API to discover authentic product URLs on popular Indian e-commerce platforms (Nykaa, Amazon India, Purplle):
     ```javascript
     const searchUrl = `https://api.firecrawl.dev/v0/search`;
     const resp = await fetch(searchUrl, {
       method: "POST",
       headers: { "Authorization": `Bearer ${FIRECRAWL_API_KEY}` },
       body: JSON.stringify({ query: `${product.brand} ${product.name} buy India` })
     });
     ```
   * **Graceful Degradation Fallback:** If Firecrawl returns an error, rate limits, or `FIRECRAWL_API_KEY` is not set, the Edge Function catches the exception and constructs a deterministic search URL:
     ```javascript
     const fallbackUrl = `https://www.amazon.in/s?k=${encodeURIComponent(`${product.brand} ${product.name}`)}`;
     ```
   * Every product card is guaranteed to provide a clickable, working purchase link regardless of external scraping API availability.

7. **Database Caching in PostgreSQL ([recommendations table](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/supabase/functions/get-recommendations/index.js)):**
   * Before returning to the client, the Edge Function saves the parsed recommendation JSON into the `recommendations` table using the service role admin client:
     ```javascript
     await supabaseAdmin
       .from("recommendations")
       .insert({
         assessment_id: assessmentId,
         user_id: userId,
         recommendation_data: finalResult
       });
     ```
   * Future visits to this assessment will now hit the cache instantly without incurring API costs.

8. **Frontend Hydration and UI Rendering ([Results.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/Results.jsx)):**
   * The Edge Function returns HTTP 200 with the JSON payload.
   * `Results.jsx` sets `recommendations = data`, stopping the loading skeleton.
   * The screen animates into view with:
     * Overall Skin Score dial indicator.
     * Identified Concerns badges with severity meters.
     * Routine Tabs switching seamlessly between Morning and Evening steps.
     * Product cards with `<FavoriteButton />` and `<ProductFeedback />` components.
     * Direct purchase CTA buttons opening e-commerce product pages in new tabs (`target="_blank" rel="noopener noreferrer"`).

---

### Flow 4: History Management and 8-Second Undo-Guarded Deletion

This flow traces how past clinical assessments and AI recommendations are retrieved relationally, displayed with historical comparisons, and deleted using an optimistic UI pattern with an 8-second safety undo countdown.

#### Step-by-Step Execution Sequence

1. **Relational Query Execution ([History.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/History.jsx)):**
   * User navigates to `/history`.
   * `History.jsx` mounts and issues a relational join query via the Supabase client:
     ```javascript
     const { data, error } = await supabase
       .from("skin_assessments")
       .select(`
         *,
         recommendations (*)
       `)
       .eq("user_id", user.id)
       .order("created_at", { ascending: false });
     ```
   * Supabase PostgreSQL translates this into an optimized SQL query:
     ```sql
     SELECT skin_assessments.*, 
            json_agg(recommendations.*) AS recommendations
     FROM skin_assessments
     LEFT JOIN recommendations ON recommendations.assessment_id = skin_assessments.id
     WHERE skin_assessments.user_id = auth.uid()
     GROUP BY skin_assessments.id
     ORDER BY skin_assessments.created_at DESC;
     ```
   * RLS policies automatically filter rows so the client only receives records owned by the authenticated user.

2. **History List and Metric Rendering ([History.jsx](file:///c:/Users/91638/Downloads/skinwise%20different%20versions/SkinWise/src/pages/History.jsx)):**
   * State updates: `setAssessments(data)` and `setLoading(false)`.
   * The component renders each historical assessment card showing:
     * Formatted date and time.
     * Skin score badge (retrieved from the nested `recommendations[0].recommendation_data.overall_score`).
     * Primary diagnosed skin type and concerns chips.
     * View Details button linking to `/results/${item.id}`.
     * Trash icon button for assessment deletion.

3. **User Clicks Delete — Optimistic Eviction & Countdown Initiation:**
   * The user clicks the trash icon on a historical assessment card with ID `targetId`.
   * Rather than freezing the UI to wait for a database round-trip, the component updates instantly:
     ```javascript
     // 1. Save reference to item being deleted
     const itemToDelete = assessments.find(a => a.id === targetId);
     
     // 2. Optimistically evict from local React state
     setAssessments(prev => prev.filter(a => a.id !== targetId));
     
     // 3. Create 8-second safety timeout
     const timeoutId = setTimeout(() => {
       commitDelete(targetId);
     }, 8000);
     
     // 4. Store references in ref to allow cancellation
     pendingDeleteRef.current = { id: targetId, item: itemToDelete, timeoutId };
     ```
   * The assessment card vanishes from the list immediately with a smooth DOM transition.

4. **Interactive Toast Display with Active Countdown:**
   * `History.jsx` triggers a toast notification containing an active 8-second countdown timer and an **"Undo"** action button:
     ```javascript
     toast({
       title: "Assessment Deleted",
       description: "Assessment removed. You have 8 seconds to undo.",
       action: (
         <ToastAction altText="Undo deletion" onClick={handleUndo}>
           Undo
         </ToastAction>
       ),
       duration: 8000
     });
     ```

5. **Branch A: User Clicks "Undo" Within the 8-Second Window:**
   * The user clicks the **Undo** button in the toast.
   * `handleUndo()` executes:
     ```javascript
     // 1. Cancel pending timer to abort database mutation
     if (pendingDeleteRef.current) {
       clearTimeout(pendingDeleteRef.current.timeoutId);
       
       // 2. Restore item back into state at its original chronological position
       setAssessments(prev => {
         const restored = [...prev, pendingDeleteRef.current.item];
         return restored.sort((a, b) => 
           new Date(b.created_at).getTime() - new Date(a.created_at).getTime()
         );
       });
       
       pendingDeleteRef.current = null;
     }
     ```
   * The card immediately re-appears in the list.
   * **Zero network requests or database mutations occurred.** The user's data was completely protected from accidental deletion.

6. **Branch B: Timer Reaches 0 Seconds Without Cancellation:**
   * The 8-second timer expires; `commitDelete(targetId)` is called:
     ```javascript
     const commitDelete = async (id: string) => {
       const { error } = await supabase
         .from("skin_assessments")
         .delete()
         .eq("id", id);
       
       if (error) {
         // Roll back optimistic delete if network failed
         setAssessments(prev => [...prev, itemToDelete]);
         toast({ title: "Delete Failed", variant: "destructive" });
       }
       pendingDeleteRef.current = null;
     };
     ```
   * PostgreSQL executes the `DELETE` query against `skin_assessments`.
   * **PostgreSQL Foreign Key Cascade:** Because the `recommendations` table defines `assessment_id UUID REFERENCES skin_assessments(id) ON DELETE CASCADE`, PostgreSQL automatically deletes the child recommendation row atomically within the same database transaction.
   * Supabase returns HTTP 204 No Content. The deletion is permanently finalized.

---

## 5. Concept Glossary

A consolidated, alphabetized cheat sheet of every core technical concept, programming term, algorithm, and architecture pattern used across the SkinWise codebase. Use this reference when preparing for technical interviews, code defense sessions, or college viva examinations.

---

### A

* **Active Ingredient:** A biologically active chemical compound within a cosmetic or pharmaceutical formulation specifically clinically proven to target a biological mechanism (e.g., *Niacinamide* for sebum regulation, *Salicylic Acid* for desquamation).
* **Aggregate Function (SQL):** A database operation that collapses multiple rows into a single summary value or composite array (e.g., `json_agg()`, `COUNT()`, `AVG()`).
* **Async / Await:** Modern JavaScript/TypeScript syntactic sugar over Promises that allows asynchronous, non-blocking network or file operations to be written in a readable, sequential style.

---

### B

* **Base64 Encoding:** A binary-to-text encoding scheme that translates raw 8-bit binary data into an ASCII string of 64 printable characters, allowing images to be transmitted directly over HTTP or JSON without requiring a separate multipart upload.
* **BlazeFace:** A lightweight, ultra-fast convolutional neural network designed by Google for real-time mobile facial detection, capable of predicting 2D face bounding boxes and 6 key facial landmarks (eyes, nose, mouth, ears) at high frame rates.
* **Bounding Box:** A rectangular coordinate envelope $(x_{\min}, y_{\min}, x_{\max}, y_{\max})$ that completely encloses an object detected by a computer vision model in a 2D image coordinate system.

---

### C

* **Client-Side Routing:** An architecture in Single Page Applications where URL changes are intercepted by client-side JavaScript (e.g., React Router) rather than causing the browser to request a new HTML document from the server.
* **Closure (JavaScript):** A fundamental programming language feature where an inner function retains access to its lexical scope (variables and parameters of the enclosing function) even after the outer function has finished executing.
* **Conditional Rendering:** A React design pattern where components dynamically render different UI branches depending on boolean conditions or the presence of data (e.g., `{loading ? <Spinner /> : <Content />}`).
* **Convolution Kernel:** A small 2D matrix of mathematical weights (e.g., $3 \times 3$) that is systematically multiplied across pixels in an image to extract features such as edges, gradients, or blur.
* **CORS (Cross-Origin Resource Sharing):** A browser security protocol using HTTP response headers (`Access-Control-Allow-Origin`) to indicate which external domains or origins are permitted to access resources on the server.
* **CSS Module vs. Utility-First CSS:** CSS Modules isolate stylesheets by generating unique scrambled class names scoped to individual files, whereas Utility-First CSS (Tailwind) provides pre-defined, atomic utility classes directly composed inside markup.

---

### D

* **Data URL (RFC 2397):** A Uniform Resource Identifier scheme (`data:[mediatype][;base64],<data>`) that enables arbitrary files (such as JPEG images) to be embedded directly as inline text strings within HTML or JSON.
* **Debouncing:** A programming technique that delays the execution of a frequently triggered function until a specified period of idle time has elapsed (e.g., waiting 300ms after a user stops typing before running a search query).
* **Deno:** A modern, secure runtime for JavaScript and TypeScript built on Rust and the V8 engine, used to power Supabase Edge Functions with native TypeScript compilation and zero-config deployment.
* **Dependency Array (`useEffect`):** The second argument passed to React's `useEffect` hook; React checks whether any values in this array changed between renders before deciding whether to rerun the effect callback.
* **Destructuring Assignment:** A JavaScript/TypeScript syntax expression that unpacks properties from objects or values from arrays into distinct, standalone variables (e.g., `const { user, loading } = useAuth();`).
* **Discrete Convolution:** The mathematical operation of sliding a finite filter kernel across a discrete pixel grid to produce a transformed feature map (used in SkinWise to compute the Laplacian edge gradient).

---

### E

* **Edge Function:** A serverless function deployed across a geographically distributed content delivery network (CDN), executing code close to the end user with minimal network latency and fast cold-start times.
* **Environment Variables (`import.meta.env`):** Configuration values provided by the host operating system or build tool to decouple sensitive credentials and API URLs from source code.
* **Error Boundary:** A React component pattern that catches JavaScript errors anywhere in its child component tree, logs the error, and displays a fallback UI instead of crashing the entire application.

---

### F

* **Firecrawl API:** A specialized web scraping and crawling service that transforms live e-commerce websites into structured markdown or JSON, handling anti-bot protections and headless browser rendering.
* **Foreign Key & Cascade Delete:** A relational database constraint where a column references the primary key of another table (`REFERENCES parent(id)`); `ON DELETE CASCADE` instructs the database to automatically delete child records whenever their parent row is deleted.

---

### G

* **Generic Types (TypeScript):** Reusable type definitions that allow components or functions to operate over a variety of types while preserving compile-time type safety (e.g., `Promise<T>`, `Array<T>`).
* **Google Gemini 2.5 Flash:** A multimodal large language model from Google trained on text, code, audio, and images, optimized for low latency and high-reasoning tasks.
* **GoTrue (Supabase Auth):** An open-source, Go-based microservice that manages user authentication, password hashing, OAuth integrations, and JWT issuance for Supabase projects.

---

### H

* **HMR (Hot Module Replacement):** A development server feature in tools like Vite that swaps out updated code modules in the running browser without performing a full page refresh, preserving application state.
* **Hook (React):** A special JavaScript function starting with `use` (e.g., `useState`, `useEffect`, `useReducer`) that lets functional components hook into React state and lifecycle features.
* **HTML5 Canvas API:** A client-side JavaScript API used to draw 2D graphics, render video frames, and extract raw pixel data (`getImageData()`) directly in the browser.

---

### I

* **Idempotency:** A property of operations where making multiple identical requests has the exact same side-effect and outcome as making a single request (e.g., HTTP `GET` and `PUT` are idempotent, whereas `POST` is not).
* **ITU-R BT.709 Luminance Standard:** An internationally recognized standard defining how red, green, and blue light components are weighted ($Y = 0.2126R + 0.7152G + 0.0722B$) to accurately match human visual perception of overall image brightness.

---

### J

* **JWT (JSON Web Token):** A compact, URL-safe standard (RFC 7519) for transmitting claims securely between two parties as a digitally signed, Base64-encoded JSON object consisting of Header, Payload, and Signature.

---

### L

* **Laplacian Operator & Variance:** A second-order differential operator that measures rapid intensity transitions in an image; the statistical variance of the Laplacian response provides a proven numerical benchmark for image sharpness and blur detection.
* **Lazy Loading (`React.lazy` / `Suspense`):** An optimization technique that splits code into separate bundles loaded on-demand only when a user navigates to the corresponding route or feature.
* **Linear Interpolation:** A method of calculating intermediate values between two distinct endpoints along a linear scale.
* **LocalStorage:** A browser Web Storage API that allows web applications to persist key-value string data permanently across browser restarts with approximately 5MB of storage capacity.
* **Lucide React:** An open-source, tree-shakeable icon library providing clean SVG icons as native React components.

---

### M

* **MediaStream API (`getUserMedia`):** A modern WebRTC browser API providing access to local audio and video hardware streams from webcams or mobile camera sensors.
* **Memoization (`useMemo`, `useCallback`):** An optimization technique that caches the result of an expensive calculation or function reference between renders, recomputing it only when specified dependencies change.
* **Middleware:** Software logic that sits between an incoming HTTP request and the final handler, commonly used to inspect headers, enforce authentication, log telemetry, or validate payloads.
* **Multimodal AI:** An artificial intelligence architecture capable of processing and synthesizing multiple distinct modalities of data simultaneously (e.g., analyzing an image alongside clinical text answers).

---

### N

* **Non-Destructive / Safe Deletion:** A UX pattern where an item is temporarily hidden or marked as deleted with an immediate undo window before physical, irreversible data destruction occurs on the database.
* **Nullish Coalescing Operator (`??`):** A logical operator that returns its right-hand operand when its left-hand operand is `null` or `undefined`, unlike `||` which treats `0`, `""`, and `false` as falsy.
* **Numeric Threshold Gating:** A validation pattern where programmatic thresholds (e.g., brightness $< 40$, blur variance $< 100$) must all be satisfied before an operation or button is unlocked.

---

### O

* **Observer / Listener Pattern:** A behavioral software pattern where an object maintains a list of dependents and notifies them automatically of any state changes (e.g., `supabase.auth.onAuthStateChange`).
* **Offscreen Canvas:** An HTML5 `<canvas>` element instantiated in memory without being attached to the visible DOM, used for background image manipulation and pixel processing.
* **Optimistic UI Update:** A user interface strategy where the UI immediately updates to reflect a requested mutation before the network request finishes, giving the impression of zero network latency.
* **Optional Chaining (`?.`):** A JavaScript operator that allows reading properties deep within an object chain without throwing a runtime `TypeError` if a reference is `null` or `undefined`.

---

### P

* **PostgreSQL:** An advanced, enterprise-grade open-source relational database management system supporting ACID transactions, complex relational queries, foreign keys, and JSON document storage.
* **Preflight Request (OPTIONS):** An HTTP `OPTIONS` request sent automatically by web browsers prior to a cross-origin request to verify whether the server permits the requested HTTP method and headers.
* **Prompt Engineering:** The practice of structuring, refining, and constraining text instructions given to an LLM to guarantee predictable, accurate, and schema-compliant outputs.
* **Provider Pattern:** A React design pattern using React Context to pass state and updater functions down a deep component hierarchy without manually passing props through intermediate components ("prop drilling").

---

### R

* **Radix UI Primitives:** An unstyled, accessible component library that handles keyboard navigation, screen reader ARIA roles, focus trapping, and modal portals, leaving visual design entirely to the developer.
* **Reducer Pattern (`useReducer`):** A state management pattern where all state changes are expressed as dispatched actions and processed by a pure function `(previousState, action) => newState`.
* **Ref (`useRef`):** A React hook that creates a persistent mutable container holding a `.current` property that survives re-renders without triggering a component re-render when mutated.
* **Relational Join:** A SQL query operation that combines fields from two or more tables based on a related common key (e.g., joining `skin_assessments` with `recommendations` on `assessment_id`).
* **Responsive Design:** A front-end design methodology using fluid grids, media queries, and flexible units to ensure interfaces adapt seamlessly across mobile phones, tablets, and desktop viewports.
* **REST API:** An architectural style for network communication relying on stateless, standard HTTP methods (`GET`, `POST`, `PUT`, `DELETE`) operating on resource URIs.
* **RLS (Row Level Security):** A database-level security engine built into PostgreSQL that evaluates row-by-row authorization policies against the requesting user's session token (`auth.uid()`).

---

### S

* **Single Page Application (SPA):** A web application that loads a single HTML page and dynamically updates content as the user interacts, eliminating page reloads and providing a desktop-like user experience.
* **State (React):** An internal data store managed within a React component that holds information that may change over time; mutations to state trigger component re-renders.
* **Structured Concurrency / Async Loops:** Managing concurrent asynchronous tasks with clear boundaries, error handling, and completion criteria rather than firing unmonitored floating promises.
* **SWC (Speedy Web Compiler):** An ultra-fast TypeScript/JavaScript compiler written in Rust, used by Vite to compile JSX and modern syntax up to 20x faster than traditional Babel.

---

### T

* **Tailwind CSS:** A utility-first CSS framework providing low-level styling classes (e.g., `flex`, `pt-4`, `text-emerald-500`) directly inside markup to create custom designs without writing ad-hoc stylesheets.
* **TensorFlow.js (`@tensorflow/tfjs`):** An open-source hardware-accelerated machine learning library for JavaScript that runs neural network models directly in the browser via WebGL.
* **Toast Notification:** A brief, non-intrusive popup message displayed to confirm an action, warn of an error, or provide time-sensitive undo functionality before fading away.
* **TypeScript Interfaces & Type Aliases:** Compile-time type contracts that define the required shapes, properties, and method signatures of JavaScript objects, catching bugs prior to production execution.

---

### U

* **UI Skeleton / Shimmer:** A placeholder wireframe animation rendered while asynchronous data is loading, maintaining visual layout stability and preventing layout shifts.
* **Upsert:** A composite database operation combining `INSERT` and `UPDATE`; if a row with the specified primary key already exists, it updates it; otherwise, it inserts a new row.
* **URI Encoding (`encodeURIComponent`):** Converting characters that are not safe in URLs into valid percent-encoded representations (e.g., spaces become `%20`) to prevent broken query parameters.

---

### V

* **Variance ($\sigma^2$):** A fundamental statistical measurement of how much a set of numbers spreads out from their average mean; in computer vision, low Laplacian variance signifies an image with flat, blurred transitions lacking sharp edges.
* **Vite:** A next-generation frontend build tool that leverages native ES modules in development for instant startup and bundles assets with Rollup for optimized production distribution.

---

### W

* **WebGL Backend (TensorFlow):** A hardware-accelerated backend that executes machine learning tensor calculations on the user's graphics processor (GPU) using WebGL shaders.
* **WebRTC / Camera Stream:** A collection of browser communication standards enabling peer-to-peer real-time audio, video, and data transmission without browser plugins.
* **Wildcard Route (`*`):** A fallback route pattern in routing libraries that catches all incoming URLs that failed to match any defined route paths, commonly used to render a 404 page.

---

### Z

* **Zod Schema Validation:** A TypeScript-first schema declaration and validation library with static type inference, used to rigorously validate input forms, API responses, and database payloads at runtime.

---

## Conclusion & Viva Preparation Checklist

When presenting or defending SkinWise in an interview or viva examination, remember these 5 high-impact talking points:

1. **Client-Side Computer Vision:** Emphasize that facial detection and quality gating run entirely on the user's browser via TensorFlow.js and WebGL. This eliminates unnecessary server load and ensures poor quality photos never consume expensive AI tokens.
2. **Deterministic Pre-Filtering:** You implemented ITU-R BT.709 photometric luminance calculation and 3x3 Laplacian variance blur detection to gate camera capture mathematically before inference.
3. **Multimodal Edge Architecture:** Highlight that Deno Edge Functions keep the `GEMINI_API_KEY` and `FIRECRAWL_API_KEY` secure on the server while combining image bytes and clinical survey answers in a single prompt.
4. **Resilient E-Commerce Discovery:** You integrated the Firecrawl API for live shopping link discovery, but engineered a deterministic fallback using Amazon.in search query encoding to ensure zero broken user experiences.
5. **Database-Enforced Security & UX:** Explain PostgreSQL Row Level Security (RLS) ensuring total multi-tenant privacy, alongside an optimistic UI deletion flow with an 8-second safe undo window.



