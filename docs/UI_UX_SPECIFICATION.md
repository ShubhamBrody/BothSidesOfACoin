# UI/UX Specification
## BothSidesOfACoin — Design System & Page Specifications

---

## 1. Design System

### Brand Identity
| Element | Value |
|---------|-------|
| **Primary Color** | `#2563EB` (Royal Blue — trust, intelligence) |
| **Left Accent** | `#3B82F6` (Blue — liberal/progressive) |
| **Right Accent** | `#EF4444` (Red — conservative) |
| **Neutral Accent** | `#8B5CF6` (Purple — balance, wisdom) |
| **Success** | `#10B981` (Emerald Green) |
| **Warning** | `#F59E0B` (Amber) |
| **Error** | `#EF4444` (Red) |
| **Background (Dark)** | `#0F172A` (Slate 900) |
| **Surface (Dark)** | `#1E293B` (Slate 800) |
| **Card (Dark)** | `#334155` (Slate 700) |
| **Text Primary (Dark)** | `#F8FAFC` (Slate 50) |
| **Text Secondary (Dark)** | `#94A3B8` (Slate 400) |
| **Background (Light)** | `#FFFFFF` |
| **Surface (Light)** | `#F8FAFC` (Slate 50) |

### Typography
| Element | Font | Size | Weight |
|---------|------|------|--------|
| **H1** | Inter | 36px / 2.25rem | 800 (ExtraBold) |
| **H2** | Inter | 30px / 1.875rem | 700 (Bold) |
| **H3** | Inter | 24px / 1.5rem | 600 (SemiBold) |
| **H4** | Inter | 20px / 1.25rem | 600 |
| **Body** | Inter | 16px / 1rem | 400 (Regular) |
| **Body Small** | Inter | 14px / 0.875rem | 400 |
| **Caption** | Inter | 12px / 0.75rem | 500 |
| **Code** | JetBrains Mono | 14px | 400 |

### Spacing Scale
Based on 4px grid: `4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80, 96, 128`

### Border Radius
| Element | Radius |
|---------|--------|
| Buttons | 8px |
| Cards | 12px |
| Modals | 16px |
| Inputs | 8px |
| Avatars | 50% (circle) |
| Badges | 16px (pill) |

### Shadows (Dark Mode)
```css
--shadow-sm: 0 1px 2px rgba(0,0,0,0.3);
--shadow-md: 0 4px 6px rgba(0,0,0,0.4);
--shadow-lg: 0 10px 15px rgba(0,0,0,0.5);
--shadow-glow-blue: 0 0 20px rgba(59,130,246,0.3);
--shadow-glow-red: 0 0 20px rgba(239,68,68,0.3);
--shadow-glow-purple: 0 0 20px rgba(139,92,246,0.3);
```

---

## 2. Animation Specifications (Framer Motion)

### Page Transitions
```typescript
// Fade + Slide Up
const pageTransition = {
  initial: { opacity: 0, y: 20 },
  animate: { opacity: 1, y: 0 },
  exit: { opacity: 0, y: -20 },
  transition: { duration: 0.3, ease: "easeOut" }
};
```

### Card Hover Effects
```typescript
const cardHover = {
  whileHover: {
    y: -4,
    boxShadow: "0 10px 30px rgba(0,0,0,0.3)",
    transition: { duration: 0.2 }
  }
};
```

### 3-Side View Reveal
```typescript
// Columns animate in from left, center, right
const leftColumn = {
  initial: { opacity: 0, x: -50 },
  animate: { opacity: 1, x: 0, transition: { delay: 0 } }
};
const centerColumn = {
  initial: { opacity: 0, y: 30 },
  animate: { opacity: 1, y: 0, transition: { delay: 0.15 } }
};
const rightColumn = {
  initial: { opacity: 0, x: 50 },
  animate: { opacity: 1, x: 0, transition: { delay: 0.3 } }
};
```

### Bias Score Gauge
```typescript
// Animated circular progress (SVG path)
const biasGauge = {
  initial: { pathLength: 0 },
  animate: { pathLength: score / 100 },
  transition: { duration: 1.5, ease: "easeInOut" }
};
```

### Skeleton Loading
```typescript
const shimmer = {
  animate: {
    backgroundPosition: ["200% 0", "-200% 0"],
  },
  transition: { duration: 1.5, repeat: Infinity }
};
```

### Chat Message Appear
```typescript
const messageAppear = {
  initial: { opacity: 0, y: 10, scale: 0.95 },
  animate: { opacity: 1, y: 0, scale: 1 },
  transition: { duration: 0.2 }
};
```

---

## 3. Responsive Breakpoints

| Breakpoint | Size | Layout |
|------------|------|--------|
| **Mobile S** | 320px | Single column, hamburger nav |
| **Mobile L** | 425px | Single column, compact cards |
| **Tablet** | 768px | 2-column where applicable |
| **Laptop** | 1024px | Full layout, sidebar |
| **Desktop** | 1440px | Full layout, wider content |
| **4K** | 2560px | Max-width container, centered |

### 3-Side View Responsive Behavior
| Breakpoint | Layout |
|------------|--------|
| Mobile (< 768px) | Tab-based: toggle Left / Neutral / Right |
| Tablet (768-1024px) | 2 columns (Left+Right) with Neutral below |
| Desktop (1024px+) | 3 columns side-by-side |

---

## 4. Page Specifications

### 4.1 Landing Page

```
┌─────────────────────────────────────────────────────────┐
│ NAVBAR: Logo  |  Features  |  Pricing  |  [Login] [SignUp] │
├─────────────────────────────────────────────────────────┤
│                                                         │
│                    HERO SECTION                          │
│     "See every side of the story"                       │
│     Subtitle: Breaking the echo chamber with AI         │
│     [Get Started Free]  [Watch Demo →]                  │
│                                                         │
│     ┌──────────┐ ┌──────────┐ ┌──────────┐             │
│     │ 🟦 Left  │ │ ⚪Neutral│ │ 🟥 Right │             │
│     │ perspect │ │ analysis │ │ perspect │             │
│     │ ive...   │ │ ...      │ │ ive...   │             │
│     └──────────┘ └──────────┘ └──────────┘             │
│     (Animated 3-Side View mockup)                       │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│     FEATURES SECTION (3x2 grid)                         │
│     ┌─────────┐ ┌─────────┐ ┌─────────┐               │
│     │3-Side   │ │Bias     │ │AI Debate│               │
│     │View     │ │Score    │ │Mode     │               │
│     └─────────┘ └─────────┘ └─────────┘               │
│     ┌─────────┐ ┌─────────┐ ┌─────────┐               │
│     │Timeline │ │Impact   │ │Smart    │               │
│     │View     │ │Analyzer │ │Feed     │               │
│     └─────────┘ └─────────┘ └─────────┘               │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│     HOW IT WORKS (3 steps, animated on scroll)          │
│     1. Pick a topic                                     │
│     2. AI analyzes from all perspectives                │
│     3. You see the full picture                         │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│     PRICING SECTION                                     │
│     ┌──────┐  ┌──────────┐  ┌──────────┐              │
│     │FREE  │  │PREMIUM   │  │ENTERPRISE│              │
│     │$0    │  │$9.99/mo  │  │$49.99/mo │              │
│     │...   │  │★ Popular │  │...       │              │
│     └──────┘  └──────────┘  └──────────┘              │
│                                                         │
├─────────────────────────────────────────────────────────┤
│                                                         │
│     TESTIMONIALS / SOCIAL PROOF                         │
│     "This changed how I consume news" — User           │
│                                                         │
├─────────────────────────────────────────────────────────┤
│     CTA: "Start seeing the full picture today"          │
│     [Get Started Free]                                  │
├─────────────────────────────────────────────────────────┤
│ FOOTER: Links | Privacy | Terms | Social | © 2026       │
└─────────────────────────────────────────────────────────┘
```

### 4.2 Dashboard

```
┌────────────────────────────────────────────────────────────────┐
│ SIDEBAR          │ MAIN CONTENT                                │
│ (collapsible)    │                                              │
│                  │ ┌──────────────────────────────────────────┐ │
│ 🏠 Dashboard     │ │ WELCOME BANNER                           │ │
│ 📈 Trending      │ │ "Good morning, John. Your bias score: 72"│ │
│ 🔍 Search        │ │ ┌─────────┐ ┌─────────┐ ┌────────────┐ │ │
│ 💬 Debates       │ │ │ Bias    │ │ Articles│ │ Debates    │ │ │
│ 📊 Bias Score    │ │ │ Score   │ │ Read    │ │ This Week  │ │ │
│ 👤 Profile       │ │ │ 🔴 72   │ │ 📰 24   │ │ 💬 5       │ │ │
│ ⚙️ Settings      │ │ └─────────┘ └─────────┘ └────────────┘ │ │
│ 💳 Upgrade       │ │                                          │ │
│                  │ ├──────────────────────────────────────────┤ │
│                  │ │ TRENDING TOPICS                          │ │
│                  │ │ ┌────────────────────────┐               │ │
│                  │ │ │ AI Regulation in EU    │ 🔥 Trending  │ │
│                  │ │ │ 24 articles │ 3 perspectives          │ │
│                  │ │ │ [View Analysis →]      │               │ │
│                  │ │ └────────────────────────┘               │ │
│                  │ │ ┌────────────────────────┐               │ │
│                  │ │ │ Global Trade Tensions  │               │ │
│                  │ │ │ ...                    │               │ │
│                  │ │ └────────────────────────┘               │ │
│                  │ │                                          │ │
│                  │ ├──────────────────────────────────────────┤ │
│                  │ │ YOUR READING DISTRIBUTION (Donut Chart)  │ │
│                  │ │ 🟦 Left: 65%  🟥 Right: 15%  ⚪ Neutral:│20%│
│                  │ │ "Consider reading more right-leaning..."│ │
│                  │ │                                          │ │
│                  │ ├──────────────────────────────────────────┤ │
│                  │ │ RECENT ACTIVITY                          │ │
│                  │ │ • Read "AI Act passes..." (Left) 2h ago │ │
│                  │ │ • Debate: "Should AI be regulated?" 1d  │ │
│                  │ │ • Read "Free market..." (Right) 2d ago  │ │
│                  │ └──────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

### 4.3 Three-Side View (MAIN USP PAGE)

```
┌───────────────────────────────────────────────────────────────────┐
│ BREADCRUMB: Dashboard > Topics > AI Regulation in EU              │
│                                                                    │
│ TOPIC HEADER                                                       │
│ "AI Regulation in the European Union"                              │
│ 📅 March 2021 - Present | 📰 24 articles | 🔥 Trending           │
│ Tags: [AI] [Regulation] [EU] [Technology]                         │
│ [Start Debate] [View Timeline]  [View Impact]                     │
│                                                                    │
├─────────────────────┬────────────────────┬────────────────────────┤
│                     │                    │                         │
│ 🟦 LEFT PERSPECTIVE │ ⚪ NEUTRAL FACTS   │ 🟥 RIGHT PERSPECTIVE   │
│ ─────────────────── │ ────────────────── │ ─────────────────────  │
│                     │                    │                         │
│ Progressive groups  │ The EU AI Act was  │ Free market advocates  │
│ argue that strong   │ proposed in April  │ warn that excessive    │
│ regulation is       │ 2021 and was       │ regulation will drive  │
│ essential to        │ officially adopted │ innovation offshore    │
│ protect citizens... │ in March 2024...   │ and harm European...   │
│                     │                    │                         │
│ KEY ARGUMENTS:      │ KEY FACTS:         │ KEY ARGUMENTS:         │
│ • AI poses risks    │ • Passed March '24 │ • Stifles innovation   │
│ • Deepens inequality│ • Risk-based tiers │ • Burdens small biz    │
│ • Protects rights   │ • Affects 450M+    │ • US/China will lead   │
│                     │                    │                         │
│ Sources:            │ Sources:           │ Sources:               │
│ The Guardian        │ Reuters            │ Financial Times        │
│ Vox                 │ AP News            │ Wall Street Journal    │
│                     │                    │                         │
│ ⚠️ AI-generated     │ ⚠️ AI-generated    │ ⚠️ AI-generated        │
│ content disclaimer  │ content disclaimer │ content disclaimer     │
│                     │                    │                         │
├─────────────────────┴────────────────────┴────────────────────────┤
│                                                                    │
│ RELATED ARTICLES (Horizontal scroll)                               │
│ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐             │
│ │ 🟦 Art 1 │ │ 🟥 Art 2 │ │ ⚪ Art 3 │ │ 🟦 Art 4 │             │
│ │ Title... │ │ Title... │ │ Title... │ │ Title... │             │
│ └──────────┘ └──────────┘ └──────────┘ └──────────┘             │
│                                                                    │
└───────────────────────────────────────────────────────────────────┘
```

### 4.4 AI Debate Mode

```
┌───────────────────────────────────────────────────────────────┐
│ AI DEBATE: "Should AI be regulated by government?"            │
│ Topic: AI Regulation in EU | [End Debate]                     │
│                                                                │
├────────────────────┬───────────────────┬──────────────────────┤
│ 🟦 LEFT AI         │ ⚪ NEUTRAL AI     │ 🟥 RIGHT AI          │
│                    │                   │                       │
│ ┌────────────────┐ │ ┌───────────────┐ │ ┌──────────────────┐ │
│ │ "Absolutely.   │ │ │ "This is a    │ │ │ "Government      │ │
│ │ Without strong │ │ │ complex issue │ │ │ regulation       │ │
│ │ regulation,    │ │ │ with valid    │ │ │ stifles the very │ │
│ │ AI will..."    │ │ │ points on     │ │ │ innovation we..."│ │
│ │ ⚠️ AI-generated│ │ │ both sides."  │ │ │ ⚠️ AI-generated  │ │
│ └────────────────┘ │ │ ⚠️ AI-generated│ │ └──────────────────┘ │
│                    │ └───────────────┘ │                       │
│ ┌────────────────┐ │ ┌───────────────┐ │ ┌──────────────────┐ │
│ │ "The EU Act    │ │ │ "Both sides   │ │ │ "Look at the     │ │
│ │ shows that     │ │ │ agree AI needs│ │ │ data: countries   │ │
│ │ thoughtful..." │ │ │ guardrails."  │ │ │ with lighter..."  │ │
│ └────────────────┘ │ └───────────────┘ │ └──────────────────┘ │
│                    │                   │                       │
├────────────────────┴───────────────────┴──────────────────────┤
│                                                                │
│ YOUR MESSAGE                                                   │
│ ┌──────────────────────────────────────────────────────┐      │
│ │ What about privacy concerns with AI?         [Send →]│      │
│ └──────────────────────────────────────────────────────┘      │
│ Target: [All] [Left only] [Right only] [Neutral only]         │
│                                                                │
│ ⚠️ AI-generated debate content may contain inaccuracies.       │
│ These are simulated perspectives, not real political stances.  │
└───────────────────────────────────────────────────────────────┘
```

### 4.5 Bias Score Dashboard

```
┌───────────────────────────────────────────────────────────┐
│ YOUR BIAS SCORE                                            │
│                                                            │
│ ┌──────────────────────────────────────────────────────┐  │
│ │                                                      │  │
│ │    ┌───────────────────┐    SCORE: 72 / 100         │  │
│ │    │                   │    Direction: LEFT-LEANING  │  │
│ │    │   ANIMATED GAUGE  │                             │  │
│ │    │   (circular SVG)  │    "You tend to consume     │  │
│ │    │    Score: 72      │    predominantly left-      │  │
│ │    │                   │    leaning content."         │  │
│ │    └───────────────────┘                             │  │
│ │                                                      │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                            │
│ READING DISTRIBUTION (Animated Donut Chart)                │
│ ┌──────────────────────────────────────────────────────┐  │
│ │  🟦 Left: 65%  │  ⚪ Neutral: 20%  │  🟥 Right: 15% │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                            │
│ BIAS TREND (Line Chart, 30 days)                          │
│ ┌──────────────────────────────────────────────────────┐  │
│ │  Score                                               │  │
│ │  100├                                                │  │
│ │   80├──────●─────●───●──────●                       │  │
│ │   60├                         ──●────●               │  │
│ │   40├                                                │  │
│ │   20├                                                │  │
│ │    0├──────┬──────┬──────┬──────┬──────┬──→ Date    │  │
│ │     Week1  Week2  Week3  Week4  Now                  │  │
│ └──────────────────────────────────────────────────────┘  │
│ Trend: IMPROVING ✅                                        │
│                                                            │
│ RECOMMENDATIONS                                            │
│ ┌──────────────────────────────────────────────────────┐  │
│ │ 💡 "Try reading conservative takes on economic       │  │
│ │    policy to broaden your perspective."              │  │
│ │    [Explore Right-Leaning Articles →]                │  │
│ │                                                      │  │
│ │ 💡 "Your neutral content reading is low. Check out   │  │
│ │    our fact-based analyses."                         │  │
│ │    [Explore Neutral Analysis →]                      │  │
│ └──────────────────────────────────────────────────────┘  │
│                                                            │
│ BADGES                                                     │
│ 🏆 Curious Reader (read 10+ topics)                       │
│ 🏆 Debate Champion (completed 5 debates)                  │
│ 🔒 Balanced Reader (score < 30 for 7 days) — locked      │
│                                                            │
└───────────────────────────────────────────────────────────┘
```

---

## 5. Component Library (shadcn/ui + Custom)

### Core Components
| Component | Source | Customization |
|-----------|--------|---------------|
| Button | shadcn/ui | Blue/Red/Purple variants for Left/Right/Neutral |
| Card | shadcn/ui | Glow border on hover for bias-colored cards |
| Dialog / Modal | shadcn/ui | Fade + scale animation |
| Input / Textarea | shadcn/ui | Focus ring color matches bias context |
| Badge | shadcn/ui | LEFT (blue), RIGHT (red), NEUTRAL (purple) |
| Tabs | shadcn/ui | Used for mobile 3-Side View |
| Avatar | shadcn/ui | For user profile and AI agent avatars |
| Tooltip | shadcn/ui | For explaining scores and metrics |
| Skeleton | shadcn/ui | Shimmer loading for all content |
| Toast | shadcn/ui | For notifications and alerts |
| Sheet | shadcn/ui | Mobile sidebar navigation |

### Custom Components
| Component | Description |
|-----------|-------------|
| `BiasGauge` | Animated circular SVG gauge showing 0-100 score |
| `PerspectiveColumn` | Single perspective card with bias-colored accents |
| `ThreeSideView` | 3-column responsive layout with animated reveal |
| `DebateChat` | Chat interface with 3 AI persona columns |
| `TypingIndicator` | Pulsing dots animation for AI typing |
| `BiasDistributionChart` | Animated donut chart (Recharts) |
| `BiasTimeline` | Line chart of bias score over time |
| `ArticleCard` | News card with bias indicator badge |
| `TimelineView` | Vertical timeline with alternating left/right events |
| `ImpactRadar` | Radar chart for multi-dimensional impact scores |
| `AIDisclaimer` | Standardized warning footer for all AI content |

### AI Disclaimer Component

```typescript
// Must appear on EVERY AI-generated content block
const AIDisclaimer = () => (
  <div className="mt-4 p-3 bg-amber-500/10 border border-amber-500/20 rounded-lg">
    <p className="text-xs text-amber-400 flex items-start gap-2">
      <AlertTriangle className="w-4 h-4 mt-0.5 flex-shrink-0" />
      <span>
        This content was generated by AI and may contain inaccuracies or reflect 
        unintended biases. Always verify information from primary sources. 
        AI-generated summaries are meant to aid understanding, not replace 
        critical thinking.
      </span>
    </p>
  </div>
);
```

---

## 6. Accessibility Requirements

| Requirement | Implementation |
|-------------|----------------|
| **Color contrast** | WCAG AA: 4.5:1 minimum for text |
| **Keyboard navigation** | All interactive elements focusable, logical tab order |
| **Screen reader** | ARIA labels on all interactive elements |
| **Reduced motion** | Respect `prefers-reduced-motion` media query |
| **Focus indicators** | Visible focus ring on all interactive elements |
| **Alt text** | All images and icons have alt text |
| **Semantic HTML** | Proper heading hierarchy, landmarks, lists |
| **Bias colors** | Never use color alone to indicate bias — always paired with text labels |
