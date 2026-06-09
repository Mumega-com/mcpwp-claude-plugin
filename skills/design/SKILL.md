---
name: design
description: Modern web design principles for building pages via mcpwp — typography, color palettes, card styling, responsive layouts, hero sections. Use before building or redesigning pages.
user-invocable: true
---

# Page Design Guide

Modern web design principles for building pages via mcpwp. $ARGUMENTS = style or project context.

## 2026 Design Principles

### Layout
- Generous whitespace: 80-120px section padding, 40-60px between elements
- Full-width sections alternating with boxed content (max 1200px)
- Mobile-first: stack to single column below 768px

### Typography
- One font family, two weights max
- Hero: 48-72px headlines, body: 18-20px
- Line height: 1.1-1.2 headlines, 1.5-1.7 body

### Cards
- Border radius: 12-16px
- Box shadow: `0 4px 20px rgba(0,0,0,0.08)`
- Hover: `translateY(-4px)` + deeper shadow, 0.3s ease
- Internal padding: 30-40px

### Color Palettes

**Dark Tech:** Primary #6366F1 | Accent #22D3EE | BG #0F172A | Text #F8FAFC
**Clean SaaS:** Primary #2563EB | Accent #F59E0B | BG #FFFFFF | Text #1E293B
**Bold Startup:** Primary #DC2626 | Accent #FBBF24 | BG #18181B | Text #FAFAFA
**Nature/Calm:** Primary #059669 | Accent #8B5CF6 | BG #F0FDF4 | Text #1C1917

### Hero Best Practices
- One headline (what it does, not what it is)
- One subheadline (1-2 sentences)
- One primary CTA (action verb)
- Dark or gradient background
- No clutter

### Feature Grid
- 3 columns desktop, 1 mobile
- Icon + title + description per card
- Left-aligned text
- Equal card heights via flex

### Responsive Card Widths
- 3-col: 30% desktop, 47% tablet, 100% mobile
- 2-col: 47% desktop, 100% mobile
- 4-col: 22% desktop, 47% tablet, 100% mobile

### Don't
- Walls of text without whitespace
- Center-align body text
- Multiple CTAs per section
- Auto-playing video backgrounds
- Stock photo heroes
- Carousel sliders for important content
