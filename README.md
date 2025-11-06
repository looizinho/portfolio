# Web App Template (Static Frontend)

Pure React + Tailwind template with shadcn/ui baked in. **Use this README as the checklist for shipping static experiences.**

> **Note:** This template includes a minimal `shared/` and `server/` directory with placeholder types to support imported templates. These are just compatibility placeholders - web-static remains a true static-only template without API functionality.

---

## 🤖 AI Development Guide

### Stack Overview
- Client-only routing powered by React + Wouter.
- Design tokens are provided through `client/src/index.css` and `tailwind.config.ts`—keep them intact.

### Component Patterns

```tsx
// Compose pages from shadcn/ui primitives
import { Button } from "@/components/ui/button";

export function Hero() {
  return (
    <section className="rounded-3xl bg-white p-10 shadow-xl">
      <h1 className="text-4xl font-bold text-slate-900">Launch Quickly</h1>
      <Button size="lg" className="mt-6">Get Started</Button>
    </section>
  );
}
```

### File Structure

```
client/
  public/         ← Static assets copied verbatim to '/'
  src/
    pages/        ← Page-level components
    components/   ← Reusable UI & shadcn/ui
    contexts/     ← React contexts
    hooks/        ← Custom React hooks
    lib/          ← Utility helpers
    App.tsx       ← Routes & top-level layout
    main.tsx      ← React entry point
    index.css     ← global style
server/         ← Placeholder for imported template compatibility
shared/         ← Placeholder for imported template compatibility
  const.ts      ← Shared constants
```

Assets placed under `client/public` are served with aggressive caching, so add a content hash to filenames (for example, `logo.3fa9b2e4.svg`) whenever you replace a file and update its references to avoid stale assets.

Files in `client/public` are available at the root of your site—reference them with absolute paths (`/logo.3fa9b2e4.svg`, `/robots.txt`, etc.) from HTML templates, JSX, or meta tags.

---

## 🎯 Development Workflow

1. **Compose pages** in `client/src/pages/`. Keep sections modular so they can be reused across routes.
2. **Share primitives** via `client/src/components/`—extend shadcn/ui when needed instead of duplicating markup.
3. **Keep styling consistent** by relying on existing Tailwind tokens (spacing, colors, typography).
4. **Fetch external data** with `useEffect` if the site needs dynamic content from public APIs.

---

## 🧱 Tailwind Safeguards

- Preserve the `@layer base` block in `client/src/index.css`; removing it breaks utilities like `border-border`.
- Do not strip values from `theme.extend` in `tailwind.config.ts`—they power the design tokens used in the UI kit.
- Stick to utility classes for responsiveness (mobile-first by default).
## ✅ Launch Checklist
- [ ] UI layout and navigation structure correct, all image src valid.
- [ ] Success + error paths verified in the browser

---

## 🎨 Frontend Best Practices (shadcn-first)

- Prefer shadcn/ui components for interactions to keep a modern, consistent look; import from `@/components/ui/*` (e.g., `button`, `card`, `dialog`).
- Compose Tailwind utilities with component variants for layout and states; avoid excessive custom CSS. Use built-in `variant`, `size`, etc. where available.
- Preserve design tokens: keep the `@layer base` rules in `client/src/index.css`. Utilities like `border-border` and `font-sans` depend on them.
- Consistent design language: use spacing, radius, shadows, and typography via tokens. Extract shared UI into `components/` for reuse instead of copy‑paste.
- Accessibility and responsiveness: keep visible focus rings and ensure keyboard reachability; design mobile‑first with thoughtful breakpoints.
- Theming: Choose dark/light theme to start with for ThemeProvider according to your design style (dark or light bg), then manage colors pallette with CSS variables in `client/src/index.css` instead of hard‑coding to keep global consistency;
- Micro‑interactions and empty states: add motion, empty states, and icons tastefully to improve quality without distracting from content.
- Navigation: Design clear and intuitive navigation structure appropriate for the app type (e.g., top/side nav for multi-page apps, breadcrumbs or contextual navigation for SPAs)'. When building dashboard-like experience, use sidebar-nav to keep all page entry easy to access.

**React component rules:**
- Never call setState/navigation in render phase → wrap in `useEffect`

---

## Common Pitfalls

### Infinite loading loops from unstable references
**Anti-pattern:** Creating new objects/arrays in render that are used as query inputs
```tsx
// ❌ Bad: New Date() creates new reference every render → infinite queries
const { data } = trpc.items.getByDate.useQuery({
  date: new Date(), // ← New object every render!
});

// ❌ Bad: Array/object literals in query input
const { data } = trpc.items.getByIds.useQuery({
  ids: [1, 2, 3], // ← New array reference every render!
});
```

**Correct approach:** Stabilize references with useState/useMemo
```tsx
// ✅ Good: Initialize once with useState
const [date] = useState(() => new Date());
const { data } = trpc.items.getByDate.useQuery({ date });

// ✅ Good: Memoize complex inputs
const ids = useMemo(() => [1, 2, 3], []);
const { data } = trpc.items.getByIds.useQuery({ ids });
```

**Why this happens:** TRPC queries trigger when input references change. Objects/arrays created in render have new references each time, causing infinite re-fetches.

### Navigation dead-ends in subpages
**Problem:** Creating nested routes without escape routes—no header nav, no sidebar, no back button.

**Solution:** Choose navigation based on app structure:
```tsx
// For dashboard/multi-section apps: Use persistent sidebar (from shadcn/ui)
import { SidebarProvider, Sidebar, SidebarContent, SidebarInset } from "@/components/ui/sidebar";

<SidebarProvider>
  <Sidebar>
    <SidebarContent>
      {/* Navigation menu items - always visible */}
    </SidebarContent>
  </Sidebar>
  <SidebarInset>
    {children}  {/* Page content */}
  </SidebarInset>
</SidebarProvider>

// For linear flows (detail pages, wizards): Use back button
import { useRouter } from "wouter";

const router = useRouter();
<div>
  <Button variant="ghost" onClick={() => router.back()}>
    ← Back
  </Button>
  <ItemDetailPage />
</div>
```

### Dark mode styling without theme configuration
**Problem:** Using dark foreground colors without setting the theme, making text invisible on default light backgrounds.

**Solution:** Set `defaultTheme="dark"` in App.tsx, then update CSS variables in `index.css`:
```tsx
// App.tsx: Set the default theme first
<ThemeProvider defaultTheme="dark">  {/* Applies .dark class to root */}
  <div className="text-foreground bg-background">
    Content  {/* Now uses dark theme CSS variables */}
  </div>
</ThemeProvider>
```

```css
/* index.css: Adjust color palette for dark theme */
.dark {
  --background: oklch(0.145 0 0);  /* Dark background */
  --foreground: oklch(0.985 0 0);  /* Light text */
  /* ... other variables ... */
}
```

---

## Core File References

`client/src/App.tsx`
```tsx
import { Toaster } from "@/components/ui/sonner";
import { TooltipProvider } from "@/components/ui/tooltip";
import { ThemeProvider } from "./contexts/ThemeContext";
import ErrorBoundary from "./components/ErrorBoundary";

// Layout Components
import Header from './components/layout/Header';
import Footer from './components/layout/Footer';

// Section Components
import HeroSection from './components/sections/HeroSection';
import StatsSection from './components/sections/StatsSection';
import ServicesSection from './components/sections/ServicesSection';
import ProcessSection from './components/sections/ProcessSection';
import ProjectsSection from './components/sections/ProjectsSection';
import TestimonialsSection from './components/sections/TestimonialsSection';
import FAQSection from './components/sections/FAQSection';
import CTASection from './components/sections/CTASection';

function App() {
  return (
    <ErrorBoundary>
      <ThemeProvider defaultTheme="dark">
        <TooltipProvider>
          <Toaster />
          <div className="App dark-theme min-h-screen">
            <Header />
            <main className="overflow-x-hidden">
              <HeroSection />
              <StatsSection />
              <ServicesSection />
              <ProcessSection />
              <ProjectsSection />
              <TestimonialsSection />
              <FAQSection />
              <CTASection />
            </main>
            <Footer />
          </div>
        </TooltipProvider>
      </ThemeProvider>
    </ErrorBoundary>
  );
}

export default App;


```

`client/src/pages/Home.tsx`
```tsx
import { Button } from "@/components/ui/button";
import { APP_LOGO, APP_TITLE } from "@/const";

/**
 * Build polished static experiences. Visit the README for the full playbook.
 * All content in this page are only for example, delete if unneeded
 */
export default function Home() {
  // If theme is switchable in App.tsx, we can implement theme toggling like this:
  // const { theme, toggleTheme } = useTheme();

  return (
    <div className="min-h-screen flex flex-col">
      <header className="w-full border-b px-4 flex items-center h-16">
        <div className="flex items-center gap-2">
          <img
            src={APP_LOGO}
            className="h-8 w-8 rounded-lg border-border bg-background object-cover"
          />
          <span className="text-xl font-bold">{APP_TITLE}</span>
        </div>
      </header>
      <main>
        Example Page
        <Button variant="default">Example Button</Button>
      </main>
    </div>
  );
}

```

`client/src/main.tsx`
```tsx
import { createRoot } from "react-dom/client";
import App from "./App";
import "./index.css";

createRoot(document.getElementById("root")!).render(<App />);

```

`client/src/index.css`
```css
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&:is(.dark *));

@theme inline {
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-popover: var(--popover);
  --color-popover-foreground: var(--popover-foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);
  --color-chart-1: var(--chart-1);
  --color-chart-2: var(--chart-2);
  --color-chart-3: var(--chart-3);
  --color-chart-4: var(--chart-4);
  --color-chart-5: var(--chart-5);
  --color-sidebar: var(--sidebar);
  --color-sidebar-foreground: var(--sidebar-foreground);
  --color-sidebar-primary: var(--sidebar-primary);
  --color-sidebar-primary-foreground: var(--sidebar-primary-foreground);
  --color-sidebar-accent: var(--sidebar-accent);
  --color-sidebar-accent-foreground: var(--sidebar-accent-foreground);
  --color-sidebar-border: var(--sidebar-border);
  --color-sidebar-ring: var(--sidebar-ring);
}

:root {
  --radius: 0.625rem;
  /* Dark Theme Colors */
  --background: #0f172a; /* Dark navy blue */
  --foreground: #f8fafc; /* Light gray/white */
  --card: #1e293b; /* Slightly lighter navy */
  --card-foreground: #f8fafc;
  --popover: #1e293b;
  --popover-foreground: #f8fafc;
  --primary: #ff6b35; /* Primary orange */
  --primary-foreground: #ffffff;
  --secondary: #8b5cf6; /* Secondary purple */
  --secondary-foreground: #ffffff;
  --muted: #334155; /* Dark gray */
  --muted-foreground: #94a3b8;
  --accent: #8b5cf6; /* Purple accent */
  --accent-foreground: #ffffff;
  --destructive: #ef4444;
  --border: #334155;
  --input: #1e293b;
  --ring: #8b5cf6;
  --chart-1: #ff6b35;
  --chart-2: #8b5cf6;
  --chart-3: #00d4aa;
  --chart-4: #ffd23f;
  --chart-5: #ff4757;
  --sidebar: #1e293b;
  --sidebar-foreground: #f8fafc;
  --sidebar-primary: #8b5cf6;
  --sidebar-primary-foreground: #ffffff;
  --sidebar-accent: #334155;
  --sidebar-accent-foreground: #f8fafc;
  --sidebar-border: #334155;
  --sidebar-ring: #8b5cf6;

  /* Custom Colors - Dark Theme */
  --primary-dark: #0f172a; /* Dark navy blue */
  --primary-navy: #1e293b; /* Slightly lighter navy */
  --accent-green: #00d4aa;
  --accent-purple: #8b5cf6; /* Secondary purple */
  --accent-orange: #ff6b35; /* Primary orange */
  --accent-yellow: #ffd23f;
  --accent-red: #ff4757;
  --accent-blue: #3b82f6;
  --accent-teal: #2ed573;
  --text-white: #ffffff;
  --text-gray: #94a3b8;
  --text-dark: #1e293b;
  --text-light: #f8fafc;
}

.dark {
  --background: #0f172a;
  --foreground: #f8fafc;
  --card: #1e293b;
  --card-foreground: #f8fafc;
  --popover: #1e293b;
  --popover-foreground: #f8fafc;
  --primary: #ff6b35;
  --primary-foreground: #ffffff;
  --secondary: #8b5cf6;
  --secondary-foreground: #ffffff;
  --muted: #334155;
  --muted-foreground: #94a3b8;
  --accent: #8b5cf6;
  --accent-foreground: #ffffff;
  --destructive: #ef4444;
  --border: #334155;
  --input: #1e293b;
  --ring: #8b5cf6;
  --chart-1: #ff6b35;
  --chart-2: #8b5cf6;
  --chart-3: #00d4aa;
  --chart-4: #ffd23f;
  --chart-5: #ff4757;
  --sidebar: #1e293b;
  --sidebar-foreground: #f8fafc;
  --sidebar-primary: #8b5cf6;
  --sidebar-primary-foreground: #ffffff;
  --sidebar-accent: #334155;
  --sidebar-accent-foreground: #f8fafc;
  --sidebar-border: #334155;
  --sidebar-ring: #8b5cf6;
}

@layer base {
  * {
    @apply border-border outline-ring/50;
  }
  body {
    @apply bg-background text-foreground;
    background-color: var(--background);
    color: var(--foreground);
    font-family: 'DM Sans', sans-serif;
  }
}

/* Custom Styles */
.hero-title {
  font-family: 'Anton', sans-serif;
  font-size: clamp(3rem, 8vw, 6rem);
  font-weight: 400;
  line-height: 1.1;
  letter-spacing: -0.02em;
}

.section-title {
  font-family: 'Anton', sans-serif;
  font-size: clamp(2rem, 5vw, 3.5rem);
  font-weight: 400;
  line-height: 1.2;
}

.section-subtitle {
  font-size: clamp(0.9rem, 2vw, 1.1rem);
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.1em;
}

.card-title {
  font-family: 'Anton', sans-serif;
  font-size: clamp(1.25rem, 3vw, 1.5rem);
  font-weight: 400;
  line-height: 1.3;
}

.bg-primary-dark {
  background-color: var(--primary-dark);
}

.bg-primary-navy {
  background-color: var(--primary-navy);
}

.bg-accent-green {
  background-color: var(--accent-green);
}

.bg-accent-purple {
  background-color: var(--accent-purple);
}

.bg-accent-orange {
  background-color: var(--accent-orange);
}

.bg-accent-yellow {
  background-color: var(--accent-yellow);
}

.bg-accent-red {
  background-color: var(--accent-red);
}

.bg-accent-blue {
  background-color: var(--accent-blue);
}

.bg-accent-teal {
  background-color: var(--accent-teal);
}

.text-accent-green {
  color: var(--accent-green);
}

.text-accent-purple {
  color: var(--accent-purple);
}

.text-accent-orange {
  color: var(--accent-orange);
}

.text-accent-yellow {
  color: var(--accent-yellow);
}

.text-accent-red {
  color: var(--accent-red);
}

.text-accent-blue {
  color: var(--accent-blue);
}

.text-accent-teal {
  color: var(--accent-teal);
}

/* Dark theme specific styles */
.dark-theme {
  background-color: var(--background);
  color: var(--foreground);
}

.dark-card {
  background-color: var(--card);
  color: var(--card-foreground);
  border: 1px solid var(--border);
}

.dark-section {
  background-color: var(--background);
}

.dark-section-alt {
  background-color: var(--primary-navy);
}

/* Logo styles */
.logo-text {
  font-family: 'Anton', sans-serif;
  font-weight: 400;
  letter-spacing: 0.02em;
}

/* Smooth scroll behavior */
html {
  scroll-behavior: smooth;
}

/* Section spacing and transitions */
section {
  position: relative;
  transition: all 0.3s ease;
}

/* Intersection observer animations */
@keyframes slideInUp {
  from {
    opacity: 0;
    transform: translateY(50px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes slideInLeft {
  from {
    opacity: 0;
    transform: translateX(-50px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

@keyframes slideInRight {
  from {
    opacity: 0;
    transform: translateX(50px);
  }
  to {
    opacity: 1;
    transform: translateX(0);
  }
}

.animate-slide-in-up {
  animation: slideInUp 0.6s ease-out;
}

.animate-slide-in-left {
  animation: slideInLeft 0.6s ease-out;
}

.animate-slide-in-right {
  animation: slideInRight 0.6s ease-out;
}

.service-card {
  transition: all 0.3s ease;
}

.service-card:hover {
  transform: scale(1.05);
  box-shadow: 0 20px 40px rgba(139, 92, 246, 0.2);
}

.fade-in {
  opacity: 0;
  transform: translateY(30px);
  transition: all 0.6s ease;
}

.fade-in.visible {
  opacity: 1;
  transform: translateY(0);
}

.slide-up {
  opacity: 0;
  transform: translateY(50px);
  transition: all 0.8s ease;
}

.slide-up.visible {
  opacity: 1;
  transform: translateY(0);
}

.decorative-element {
  position: absolute;
  pointer-events: none;
}

.gradient-text {
  background: linear-gradient(135deg, var(--accent-purple), var(--accent-orange));
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

.btn-primary {
  background: linear-gradient(135deg, var(--accent-orange), var(--accent-red));
  color: white;
  padding: 12px 24px;
  border-radius: 50px;
  font-weight: 600;
  transition: all 0.3s ease;
  border: none;
  cursor: pointer;
}

.btn-primary:hover {
  transform: scale(1.02);
  box-shadow: 0 10px 20px rgba(255, 107, 53, 0.3);
}

.metric-card {
  border-radius: 20px;
  padding: 2rem;
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
  background-color: var(--card);
  border: 1px solid var(--border);
}

.metric-card:hover {
  transform: rotate(2deg) scale(1.02);
  box-shadow: 0 20px 40px rgba(139, 92, 246, 0.2);
}

.process-card {
  border-radius: 16px;
  padding: 2rem;
  margin-bottom: 1rem;
  transition: all 0.3s ease;
  background-color: var(--card);
  border: 1px solid var(--border);
}

.process-card:hover {
  transform: translateY(-5px);
  box-shadow: 0 20px 40px rgba(139, 92, 246, 0.2);
}

.process-number {
  font-size: 5rem;
  font-weight: 900;
  line-height: 1;
}

@media (max-width: 768px) {
  .hero-title {
    font-size: clamp(2rem, 6vw, 3rem);
  }
  
  .section-title {
    font-size: clamp(1.5rem, 4vw, 2.5rem);
  }
  
  .process-number {
    font-size: 3rem;
  }
}


```

