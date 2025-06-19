# AI Web Development Rules & Guidelines

## 🎯 Core Principles

### Technology Stack Requirements
- **Framework**: Astro 5.x+ (Static Site Generation)
- **Styling**: TailwindCSS 4.x+ (Utility-first CSS)
- **Deployment**: GitHub Actions → GitHub Pages
- **Hosting**: GitHub Pages from repository root
- **Domain**: Custom domain support via CNAME

### Development Philosophy
1. **Component-First**: Break everything into small, reusable components
2. **Performance-First**: Optimize for speed and Core Web Vitals
3. **SEO-First**: Built-in optimization for search engines
4. **Privacy-First**: No unnecessary third-party scripts
5. **Mobile-First**: Responsive design from the start

---

## 📁 File Structure Rules

### Required Directory Structure
```
project-root/
├── .github/
│   └── workflows/
│       └── deploy.yml           # GitHub Actions deployment
├── public/                      # Static assets (auto-copied to root)
│   ├── CNAME                   # Custom domain (if applicable)
│   ├── favicon.svg             # Site icon
│   ├── robots.txt              # SEO crawling rules
│   └── images/                 # Optimized images
├── src/
│   ├── components/             # Reusable UI components
│   │   ├── Navigation.astro    # Site navigation
│   │   ├── Hero.astro         # Hero sections
│   │   ├── Footer.astro       # Site footer
│   │   └── [Feature].astro    # Feature-specific components
│   ├── layouts/
│   │   └── Layout.astro       # Base page layout
│   ├── pages/
│   │   ├── index.astro        # Homepage
│   │   └── [...].astro        # Additional pages
│   └── styles/
│       └── global.css         # Global styles (if needed)
├── docs/                       # Documentation (ignored in production)
├── astro.config.mjs           # Astro configuration
├── tailwind.config.mjs        # TailwindCSS configuration
├── package.json               # Dependencies and scripts
└── README.md                  # Project documentation
```

### File Naming Conventions
- **Components**: PascalCase (e.g., `HeroSection.astro`, `CallToAction.astro`)
- **Pages**: kebab-case (e.g., `index.astro`, `about-us.astro`)
- **Assets**: kebab-case (e.g., `app-screenshot.png`, `hero-background.jpg`)
- **Docs**: SCREAMING_SNAKE_CASE (e.g., `FEATURES.md`, `COPY_VARIATIONS.md`)

---

## 🏗️ Component Architecture Rules

### Component Size Limits
- **Maximum**: 150 lines per component
- **Ideal**: 50-100 lines per component
- **Break down if**: Component handles multiple concerns

### Component Responsibility Rules
1. **Single Responsibility**: Each component does ONE thing well
2. **Self-Contained**: No external dependencies except props
3. **Reusable**: Can be used in multiple contexts
4. **Accessible**: Proper ARIA labels and semantic HTML

### Component Structure Template
```astro
---
// Component: ComponentName.astro
// Purpose: Brief description of component function
// Props: List expected props with types

export interface Props {
  title: string;
  description?: string;
  variant?: 'primary' | 'secondary';
}

const { title, description, variant = 'primary' } = Astro.props;
---

<section class="component-wrapper" data-component="component-name">
  <!-- Semantic HTML structure -->
  <h2 class="text-2xl font-bold">{title}</h2>
  {description && <p class="text-gray-600">{description}</p>}
</section>

<style>
  /* Component-specific styles only if absolutely necessary */
  /* Prefer TailwindCSS classes */
</style>
```

### When to Create New Components
✅ **Create component when:**
- Content section > 30 lines
- Repeated UI patterns (nav, footer, cards)
- Interactive elements (forms, buttons, modals)
- Complex layouts (grids, hero sections)

❌ **Don't create component for:**
- Single-use text blocks < 20 lines
- Simple HTML tags with no logic
- One-off styling variations

---

## ⚙️ Astro Configuration Rules

### Required astro.config.mjs Setup
```javascript
// @ts-check
import { defineConfig } from 'astro/config';
import tailwindcss from '@tailwindcss/vite';

export default defineConfig({
  // Site configuration
  site: 'https://yourdomain.com', // or 'https://username.github.io'
  base: '/', // Use '/' for custom domain, '/repo-name' for GitHub subdomain
  
  // Output configuration
  output: 'static', // Always static for GitHub Pages
  outDir: './dist',
  
  // Performance optimizations
  vite: {
    plugins: [tailwindcss()],
    build: {
      cssCodeSplit: false, // Bundle CSS for better loading
      rollupOptions: {
        output: {
          assetFileNames: 'assets/[name].[hash][extname]'
        }
      }
    }
  },
  
  // SEO and meta
  compressHTML: true,
  
  // Image optimization
  image: {
    domains: ['yourdomain.com'],
    remotePatterns: [{ protocol: 'https' }]
  }
});
```

### Base Layout Requirements
```astro
---
// layouts/Layout.astro
export interface Props {
  title: string;
  description: string;
  image?: string;
  noindex?: boolean;
}

const { title, description, image, noindex = false } = Astro.props;
const canonicalURL = new URL(Astro.url.pathname, Astro.site);
const socialImage = image || '/images/og-default.jpg';
---

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  
  <!-- SEO Meta Tags -->
  <title>{title}</title>
  <meta name="description" content={description}>
  <link rel="canonical" href={canonicalURL}>
  {noindex && <meta name="robots" content="noindex">}
  
  <!-- Open Graph -->
  <meta property="og:type" content="website">
  <meta property="og:url" content={Astro.url}>
  <meta property="og:title" content={title}>
  <meta property="og:description" content={description}>
  <meta property="og:image" content={new URL(socialImage, Astro.url)}>
  
  <!-- Twitter Card -->
  <meta property="twitter:card" content="summary_large_image">
  <meta property="twitter:url" content={Astro.url}>
  <meta property="twitter:title" content={title}>
  <meta property="twitter:description" content={description}>
  <meta property="twitter:image" content={new URL(socialImage, Astro.url)}>
  
  <!-- Icons -->
  <link rel="icon" type="image/svg+xml" href="/favicon.svg">
  
  <!-- Fonts (if using external) -->
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
</head>
<body>
  <slot />
</body>
</html>
```

---

## 🚀 GitHub Actions Deployment Rules

### Required .github/workflows/deploy.yml
```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Build site
        run: npm run build
      
      - name: Setup Pages
        uses: actions/configure-pages@v4
      
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: './dist'

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Required package.json Scripts
```json
{
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "preview": "astro preview",
    "check": "astro check"
  }
}
```

---

## 🎨 Styling Rules (TailwindCSS)

### Design System Requirements
```javascript
// tailwind.config.mjs
export default {
  content: ['./src/**/*.{astro,html,js,jsx,md,mdx,svelte,ts,tsx,vue}'],
  theme: {
    extend: {
      // Define brand colors
      colors: {
        primary: {
          50: '#...',
          500: '#...',  // Main brand color
          900: '#...'
        }
      },
      // Define typography scale
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
      },
      // Define spacing scale
      spacing: {
        '18': '4.5rem',
        '88': '22rem'
      }
    }
  }
}
```

### CSS Class Organization Rules
1. **Layout first**: `flex`, `grid`, `absolute`, etc.
2. **Sizing**: `w-full`, `h-64`, `max-w-4xl`, etc.
3. **Spacing**: `p-4`, `m-8`, `space-y-6`, etc.
4. **Typography**: `text-lg`, `font-bold`, `leading-tight`, etc.
5. **Colors**: `bg-blue-500`, `text-gray-900`, etc.
6. **Effects**: `shadow-lg`, `hover:scale-105`, etc.

### Responsive Design Rules
```astro
<!-- Always mobile-first, then desktop -->
<div class="
  text-lg          /* Mobile: large text */
  md:text-xl       /* Tablet: extra large */
  lg:text-2xl      /* Desktop: 2xl */
  
  p-4             /* Mobile: padding 1rem */
  md:p-6          /* Tablet: padding 1.5rem */
  lg:p-8          /* Desktop: padding 2rem */
">
```

---

## 🖼️ Asset Management Rules

### Image Optimization Requirements
- **Format**: WebP primary, PNG/JPG fallback
- **Sizes**: Multiple sizes for responsive images
- **Compression**: 80-85% quality for photos
- **Location**: All images in `/public/images/`

### Asset Naming Convention
```
public/
├── images/
│   ├── hero-background.webp        # Descriptive, kebab-case
│   ├── hero-background@2x.webp     # Retina version
│   ├── features/                   # Organized by section
│   │   ├── feature-screenshot-1.webp
│   │   └── feature-screenshot-2.webp
│   └── og/                         # Social media images
│       └── og-default.jpg
```

### Performance Targets
- **Hero images**: < 100KB
- **Feature images**: < 50KB
- **Icons**: < 10KB (prefer SVG)
- **Total page weight**: < 500KB

---

## 📊 SEO & Performance Rules

### Required Meta Tags (in Layout.astro)
```astro
<!-- Essential SEO -->
<title>{title} | Brand Name</title>
<meta name="description" content={description}>
<link rel="canonical" href={canonicalURL}>

<!-- Open Graph -->
<meta property="og:title" content={title}>
<meta property="og:description" content={description}>
<meta property="og:image" content={socialImage}>
<meta property="og:url" content={canonicalURL}>

<!-- Performance -->
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link rel="preconnect" href="https://fonts.googleapis.com">
```

### Performance Optimization Rules
1. **Lazy load images**: Use `loading="lazy"` for non-hero images
2. **Preload critical assets**: Fonts, hero images
3. **Minimize JavaScript**: Astro components over client-side JS
4. **Compress assets**: WebP images, minified CSS/JS

### Core Web Vitals Targets
- **LCP**: < 2.5 seconds
- **FID**: < 100 milliseconds  
- **CLS**: < 0.1
- **Lighthouse Score**: 90+ on all metrics

---

## 🔧 Development Workflow Rules

### Branch Strategy
- **main**: Production-ready code only
- **feature/**: New features (`feature/hero-section`)
- **fix/**: Bug fixes (`fix/mobile-navigation`)

### Commit Message Format
```
type(scope): description

feat(hero): add new call-to-action button
fix(nav): resolve mobile menu overflow
docs(readme): update installation instructions
```

### Pre-deployment Checklist
- [ ] `npm run build` completes without errors
- [ ] All images optimized and < size limits
- [ ] SEO meta tags populated
- [ ] Mobile responsive design tested
- [ ] Lighthouse score 90+ on all metrics
- [ ] No console errors in browser

### Local Development Commands
```bash
# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview

# Type checking (if using TypeScript)
npm run check
```

---

## 🚫 Anti-Patterns to Avoid

### File Organization
❌ **Don't:**
- Put everything in one large file
- Mix concerns in single components
- Use inline styles instead of TailwindCSS
- Hardcode URLs or paths

### Performance
❌ **Don't:**
- Load unnecessary JavaScript
- Use unoptimized images
- Include unused CSS/JS
- Block rendering with synchronous scripts

### SEO
❌ **Don't:**
- Missing meta descriptions
- Duplicate title tags
- Missing alt text on images
- Broken internal links

### Accessibility
❌ **Don't:**
- Missing ARIA labels
- Poor color contrast
- Keyboard navigation issues
- Missing semantic HTML

---

## ✅ Success Criteria

### Technical Requirements
- [ ] Lighthouse score 90+ (Performance, Accessibility, Best Practices, SEO)
- [ ] Mobile-first responsive design
- [ ] Zero console errors
- [ ] Fast deployment via GitHub Actions

### Code Quality
- [ ] Components < 150 lines each
- [ ] Clear file organization
- [ ] Consistent naming conventions
- [ ] Proper TypeScript types (if applicable)

### User Experience
- [ ] Fast loading (< 3 seconds)
- [ ] Intuitive navigation
- [ ] Clear call-to-actions
- [ ] Accessible design

---

## 📚 Reference Links

- [Astro Documentation](https://docs.astro.build/)
- [TailwindCSS Documentation](https://tailwindcss.com/docs)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Web.dev Performance Guide](https://web.dev/performance/)

---

*Last Updated: December 2024*
*Applies to: Astro 5.x, TailwindCSS 4.x, GitHub Pages deployment* 