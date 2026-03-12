---
name: nextjs-landing-page
description: Use when implementing landing pages with Next.js App Router and Tailwind CSS. Covers project structure, component patterns, responsive design, image optimization, font loading, accessibility, SEO, and performance. Triggers on Next.js landing page implementation, Tailwind setup, responsive design, Core Web Vitals optimization.
---

# Next.js Landing Page Patterns

Production patterns for building landing pages with Next.js App Router and Tailwind CSS. Optimized for static generation, brand compliance, and Core Web Vitals.

## Project Structure

```
src/
  app/
    layout.tsx          # Root layout — fonts, metadata, global styles
    page.tsx            # Landing page — composes section components
    globals.css         # Tailwind directives + any custom CSS
  components/
    ui/                 # Shared primitives (Button, Container, Typography)
    sections/           # Page sections (Hero, Features, Testimonials, CTA, Footer)
  lib/                  # Utilities (cn helper, constants)
public/
  images/               # Optimized static images
  fonts/                # Brand font files (if not using Google Fonts)
brand-context.json      # Brand design values (consumed by tailwind.config.ts)
tailwind.config.ts      # Extended with brand design tokens
```

## Tailwind Brand Token Setup

Configure `tailwind.config.ts` with values from `brand-context.json`:

```typescript
import type { Config } from "tailwindcss";

const config: Config = {
  content: ["./src/**/*.{ts,tsx}"],
  theme: {
    extend: {
      colors: {
        brand: {
          primary: "var(--color-brand-primary)",
          secondary: "var(--color-brand-secondary)",
          accent: "var(--color-brand-accent)",
        },
        neutral: {
          dark: "var(--color-neutral-dark)",
          medium: "var(--color-neutral-medium)",
          light: "var(--color-neutral-light)",
        },
      },
      fontFamily: {
        heading: ["var(--font-heading)", "sans-serif"],
        body: ["var(--font-body)", "sans-serif"],
      },
    },
  },
};

export default config;
```

Set CSS custom properties in `globals.css` or the root layout to connect brand-context values to Tailwind tokens.

## Font Loading

Always use `next/font` for optimal loading:

```typescript
// src/app/layout.tsx
import localFont from "next/font/local";

const headingFont = localFont({
  src: "../../public/fonts/BrandHeading-Variable.woff2",
  variable: "--font-heading",
  display: "swap",
});

const bodyFont = localFont({
  src: [
    { path: "../../public/fonts/BrandBody-Regular.woff2", weight: "400" },
    { path: "../../public/fonts/BrandBody-Medium.woff2", weight: "500" },
    { path: "../../public/fonts/BrandBody-Bold.woff2", weight: "700" },
  ],
  variable: "--font-body",
  display: "swap",
});

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en" className={`${headingFont.variable} ${bodyFont.variable}`}>
      <body className="font-body text-neutral-dark">{children}</body>
    </html>
  );
}
```

## Section Wrapper Pattern

Every page section uses a consistent wrapper for padding, max-width, and responsive spacing:

```typescript
// src/components/ui/Container.tsx
interface ContainerProps {
  children: React.ReactNode;
  className?: string;
  as?: "section" | "div" | "header" | "footer";
  id?: string;
}

export function Container({
  children,
  className = "",
  as: Element = "section",
  id,
}: ContainerProps) {
  return (
    <Element id={id} className={`px-4 sm:px-6 lg:px-8 ${className}`}>
      <div className="mx-auto max-w-7xl">{children}</div>
    </Element>
  );
}
```

## Page Composition

The landing page composes section components:

```typescript
// src/app/page.tsx
import { Hero } from "@/components/sections/Hero";
import { Features } from "@/components/sections/Features";
import { Testimonials } from "@/components/sections/Testimonials";
import { CallToAction } from "@/components/sections/CallToAction";
import { Footer } from "@/components/sections/Footer";

export default function LandingPage() {
  return (
    <main>
      <Hero />
      <Features />
      <Testimonials />
      <CallToAction />
      <Footer />
    </main>
  );
}
```

## Image Optimization

Always use `next/image`:

```typescript
import Image from "next/image";

// Hero image — above the fold, use priority
<Image
  src="/images/hero.webp"
  alt="Descriptive alt text"
  width={1200}
  height={630}
  priority
  className="w-full h-auto"
/>

// Below-fold images — lazy load (default)
<Image
  src="/images/feature.webp"
  alt="Descriptive alt text"
  width={600}
  height={400}
  className="rounded-lg"
/>
```

**Rules:**
- Use `priority` only for above-the-fold images (hero, logo)
- Provide explicit `width` and `height` to prevent CLS
- Use WebP or AVIF format for photos
- SVG for icons and logos
- Always include descriptive `alt` text

## Server vs. Client Components

Landing pages should be almost entirely server components:

**Server (default)** — all static content, sections, layout:
```typescript
// No "use client" directive needed
export function Hero() {
  return (
    <section className="py-20 bg-brand-primary">
      <h1 className="font-heading text-5xl">Static headline</h1>
    </section>
  );
}
```

**Client** — only for interactivity:
```typescript
"use client";
// Mobile menu toggle, form validation, carousels, scroll-triggered animations
```

Keep client components small and leaf-level. Never make a section wrapper a client component.

## Responsive Design

Mobile-first with Tailwind breakpoints:

```typescript
<h1 className="text-3xl sm:text-4xl lg:text-5xl xl:text-6xl font-heading">
  Headline
</h1>

<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 lg:gap-8">
  {features.map(feature => (
    <FeatureCard key={feature.title} {...feature} />
  ))}
</div>
```

**Breakpoints:**
- Default: mobile (< 640px)
- `sm`: 640px+
- `md`: 768px+
- `lg`: 1024px+
- `xl`: 1280px+

## Accessibility Patterns

```typescript
// Skip navigation
<a href="#main-content" className="sr-only focus:not-sr-only focus:absolute focus:top-4 focus:left-4 bg-white px-4 py-2 z-50">
  Skip to main content
</a>

// Semantic sections with labels
<section aria-labelledby="features-heading">
  <h2 id="features-heading">Features</h2>
</section>

// Interactive elements
<button
  type="button"
  aria-expanded={isOpen}
  aria-controls="mobile-menu"
  onClick={() => setIsOpen(!isOpen)}
>
  <span className="sr-only">{isOpen ? "Close menu" : "Open menu"}</span>
  <MenuIcon aria-hidden="true" />
</button>
```

**Rules:**
- Every `<img>` has `alt` text (decorative images use `alt=""` with `aria-hidden="true"`)
- Heading hierarchy: one `<h1>`, sequential `<h2>`, `<h3>` etc.
- Color contrast: 4.5:1 minimum for body text, 3:1 for large text
- Focus indicators visible on all interactive elements
- Forms have associated labels

## SEO Metadata

```typescript
// src/app/layout.tsx
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Page Title — Brand Name",
  description: "Compelling meta description under 160 characters.",
  openGraph: {
    title: "Page Title — Brand Name",
    description: "Description for social sharing.",
    images: [{ url: "/images/og-image.jpg", width: 1200, height: 630 }],
    type: "website",
  },
  twitter: {
    card: "summary_large_image",
  },
};
```

## Performance Targets

- **LCP** (Largest Contentful Paint): < 2.5s
- **CLS** (Cumulative Layout Shift): < 0.1
- **INP** (Interaction to Next Paint): < 200ms

Key strategies:
- Static generation (no runtime data fetching)
- `priority` on hero image
- `font-display: swap` on all fonts
- Explicit dimensions on images and media
- Minimize client-side JavaScript
