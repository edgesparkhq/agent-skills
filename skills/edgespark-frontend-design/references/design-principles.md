# EdgeSpark Frontend Design Principles

This reference captures a layered frontend design system for EdgeSpark UI work.

## Contents

- Why This Works
- Layered Standard
- Direction First
- Redesign Diagnosis
- Layout And Spacing
- Typography
- Color
- Visual Treatments
- Motion And Interaction
- Landing Pages
- Apps
- Content Authenticity
- Visual Asset Role
- Utility Copy For Product UI
- Responsive And Accessibility Baseline
- Optimization Order
- Execution Style
- Anti-Patterns
- Litmus Checks

## Why This Works

The core insight is not one isolated taste rule. The quality comes from layered prompt pressure:

1. a base web-design system
2. an aesthetic boost layer
3. a concrete anti-pattern list that blocks generic output

That combination pushes the model away from "safe but bland" UI. Keep those layers intact when using this skill.

## Layered Standard

### Layer 1: Base Web System

Use these as the non-negotiable foundation:

- context-first framing
- 12-column responsive grid
- 8-point spacing system
- semantic design tokens
- semantic color roles such as `bg`, `surface`, `text`, `brand`
- WCAG 2.1 AA contrast
- responsive mobile and desktop behavior
- scroll-triggered entry and interactive feedback
- token-based light and dark mode support when relevant

### Layer 2: Aesthetic Lift

Use these to push the design above generic UI:

- soft neutral or premium muted palettes
- asymmetry and dynamic tension
- dramatic type contrast
- generous whitespace
- refined shadows, blur, texture, and layered depth
- stronger motion choreography
- premium, eye-friendly visual restraint

### Layer 3: Anti-Generic Constraints

Use the `Anti-Patterns` section as a hard gate before shipping. Those bans exist to stop the model from falling back to average frontend tropes.

## Direction First

Start every design task by fixing the product context:

- user goal
- audience
- primary tasks
- key content types
- success metric
- desired emotional response

Translate that into a minimal user journey and let it shape hierarchy, layout, copy, and interaction. Do not start from generic components and hope the result becomes distinctive later.

Before coding, commit to:

- a strong aesthetic direction
- a clear differentiator
- a single unforgettable element

Good directions are concrete and opinionated: editorial, luxury, industrial, brutalist, playful, retro-futuristic, organic, maximalist, or restrained minimalism.

## Redesign Diagnosis

When improving an existing UI, inspect the current experience before changing code:

1. Analyze target audience and use cases.
2. Identify current brand positioning.
3. Assess the intended emotional experience.
4. Check whether the palette is visually fatiguing.
5. Check whether the layout feels traditional or monotonous.
6. Identify what modern elements are missing.
7. Decide whether the interface feels too static and needs motion.

## Layout And Spacing

- Start with composition, not components.
- Default to cardless layouts. Prefer sections, columns, dividers, lists, and media blocks before reaching for cards.
- Use a 12-column responsive grid.
- Use semantic tokens for container width, columns, gutters, and margins.
- Let breakpoints drive container widths and gutters.
- Keep outer margins symmetric even when internal composition becomes asymmetric.
- Default to generous whitespace using an 8-point spacing scale.

To avoid generic layouts:

- break symmetry
- create dynamic visual tension
- use overlap and depth
- let some elements break the grid on purpose
- introduce diagonal flow or staggered rhythm where appropriate

Generous negative space often creates a more premium feel than filling every gap. If you choose density, make it controlled and intentional rather than cluttered.

## Typography

Typography should do most of the work of making the page feel designed.

- Avoid Inter, Roboto, Arial, and generic system-font defaults.
- Use no more than two typefaces unless there is a clear reason to expand the system.
- Pair a characterful display face with a refined body face.
- Use a semantic type scale: display, heading, subtitle, body, label.
- Tighten tracking for large display text when appropriate.
- Loosen tracking for all-caps labels and microcopy.
- Create dramatic size contrast when the composition calls for it.

Copy rules:

- keep headlines to one idea
- let the headline carry the meaning
- prefer concrete nouns or verb-first phrasing
- supporting copy should usually be one short sentence
- keep CTAs to one or two words
- keep navigation labels short and parallel
- avoid empty marketing filler
- cut repetition between sections
- do not include prompt language or design commentary in the UI
- if deleting 30 percent of the copy improves the page, keep deleting

Practical limits:

- headings: usually no more than 8 to 12 words
- nav labels: usually no more than 2 to 3 words

## Color

Build color from semantic roles, not scattered hex values.

- Define `bg`, `surface`, `text`, `brand`, and intent-state roles.
- Generate tonal steps from a seed color when possible.
- If no brand color exists, begin from a neutral base.
- Use one accent color by default unless the product already has a strong multi-color system.
- Use CSS variables or the project's token system.

Palette priority:

1. Soft neutral tones
2. Premium combinations such as off-white, deep gray, warm gray, muted blue, olive, or desaturated green
3. Subtle tonal variation for depth

Avoid by default:

- purple-heavy palettes
- neon accents
- high-chroma gradients
- overly vibrant schemes that create visual fatigue

If the brand truly requires purple or intense color, reduce saturation and re-check contrast.

## Visual Treatments

Use effects to support hierarchy, not as decoration for its own sake.

- Use whitespace, alignment, scale, cropping, and contrast before adding chrome.
- soft layered shadows
- restrained backdrop blur
- tasteful gradients
- hairline borders
- subtle noise or grain
- geometric patterns
- layered transparencies
- material textures

Effects should be consistent, performant, and accessible. Do not place text over heavy blur or low-contrast artwork.

## Motion And Interaction

Motion should make the interface feel alive without becoming noisy.

For visually led work, ship at least 2 to 3 intentional motions:

- one entrance sequence in the hero
- one scroll-linked, sticky, or depth effect
- one hover, reveal, or layout transition that sharpens affordance

Prefer:

- a single coherent page-load sequence
- scroll-triggered entry with one subtle effect such as fade plus slight translate
- staggered timing for grouped elements
- refined hover states
- smooth loading states when the product needs them
- immediate press and release feedback
- smooth page transitions when the app structure supports them
- occasional delightful surprise details when they do not interrupt use

Guidelines:

- prefer CSS animations by default
- use motion to guide attention and clarify hierarchy
- trigger scroll entry with intersection observers when appropriate
- for React work, prefer Framer Motion or Motion plus animation tokens
- use Framer Motion or Motion especially for section reveals, shared layout transitions, scroll-linked transforms, sticky storytelling, narrative carousels, and menus, drawers, or modal presence effects
- avoid hardcoded one-off timings inside components
- keep animation tokens consistent across the UI
- layered timing is better than many unrelated effects
- make motion noticeable in a quick recording, smooth on mobile, fast, and restrained
- remove it if it is ornamental only
- surprise details are welcome only when they do not interrupt use

## Landing Pages

### Base Structure

Default to a four-section sequence:

1. Hero: brand or product, promise, CTA, and one dominant visual.
2. Support: one concrete feature, offer, or proof point.
3. Detail: atmosphere, workflow, product depth, or story.
4. Final CTA: convert, start, visit, or contact.

Viewport budget:

- If a sticky or fixed header is present, it counts against the hero. The combined header plus hero content must fit within the initial viewport at common desktop and mobile sizes.
- When using `100vh` or `100svh` heroes, subtract persistent UI chrome or overlay the header instead of stacking it in normal flow.

### Aesthetic Lift

- Treat the first viewport as a poster, not a document.
- One composition only.
- Full-bleed image or dominant visual plane by default. No inherited page gutters or framed container on the hero itself; constrain only the inner text and action column.
- Make the brand or product name the loudest text.
- Brand first, headline second, body third, CTA fourth.
- Keep the text column narrow and anchored to a calm area of the image.
- All text over imagery must maintain strong contrast and clear tap targets.
- Headlines roughly 2-3 lines on desktop, readable in one glance on mobile.

### Anti-Generic Constraints

- No hero cards, stat strips, logo clouds, pill soup, or floating dashboards by default.
- No boxed or center-column hero when the brief calls for full bleed.
- No split-screen hero unless the text sits on a calm, unified side.

If the first viewport still works after removing the image, the image is too weak. If the brand disappears after hiding the nav, the hierarchy is too weak.

## Apps

### Base Structure

Organize around:

- primary workspace
- navigation
- secondary context or inspector
- one clear accent for action or state

Start with the working surface itself rather than a decorative hero.

### Aesthetic Lift

Default to restrained app UI:

- calm surface hierarchy
- strong typography and spacing
- few colors
- dense but readable information
- minimal chrome

If a panel can become plain layout without losing meaning, remove the card treatment.

### Anti-Generic Constraints

- No dashboard-card mosaics.
- No thick borders on every region.
- No decorative gradients behind routine product UI.
- No multiple competing accent colors.
- No ornamental icons that do not improve scanning.
- Cards only when the card is the interaction.

## Content Authenticity

Authentic content quality is part of the design standard.

Replace:

- placeholder images
- lorem ipsum
- obvious sample text
- incomplete information blocks

Prefer:

- at least one strong, real-looking image for brands, venues, editorial pages, and lifestyle products
- real, on-theme photography
- in-situ photography over abstract gradients or fake 3D objects when imagery is carrying narrative weight
- relevant product or industry imagery
- believable demo content
- real information and data when the section needs specificity
- copy that matches the product tone and audience

When visual content matters, default to real imagery rather than placeholders. This applies especially to:

- landing pages
- hero sections
- feature areas
- galleries
- portfolios
- testimonial bands
- showcase cards

Default search policy for visual work:

- proactively search for real imagery when the page has meaningful visual storytelling
- search hero, feature, background, gallery, testimonial, and showcase assets by section purpose
- use stock-image search first for generic imagery
- use entity-aware image search for recognizable brands, products, or people when needed
- use design inspiration search when visual references are needed
- use placeholders only in the limited cases listed below

Placeholder content is acceptable only when:

- the task is mostly logic or backend
- the UI is an internal tool where visuals are secondary
- the user explicitly wants placeholders
- the work is a rapid prototype where polish is intentionally deferred

When selecting imagery, check:

- relevance to the section purpose
- believable human subjects
- professional composition
- adequate resolution
- consistency across the page
- cultural fit for the product context
- whether the image avoids an obvious stock-photo feel
- whether the crop preserves a stable tonal area for text
- whether embedded signage, logos, or typographic clutter fight the UI
- whether the image introduces built-in frames, cards, panels, or split compositions that constrain the layout
- whether multiple images would work better than a single collage

## Visual Asset Role

Use images to:

- clarify meaning
- strengthen brand
- create hierarchy

When brand assets are missing, create or source on-theme, high-quality visuals instead of leaving the interface visually hollow.
The first viewport needs a real visual anchor when the brief is image-led. Decorative texture alone is not enough.

## Utility Copy For Product UI

When the work is a dashboard, app surface, admin tool, or operational workspace, default to utility copy over marketing copy.

Prefer:

- orientation, status, and action over promise, mood, or brand voice
- starting with the working surface: KPIs, charts, filters, tables, status, or task context
- headings that say what the area is or what the user can do there
- supporting text that explains scope, behavior, freshness, or decision value in one sentence
- examples: "Selected KPIs", "Plan status", "Search metrics", "Top segments", "Last sync"

Avoid on product surfaces:

- aspirational hero lines or metaphors
- campaign-style language
- hero sections unless explicitly requested
- sentences that could appear in a homepage hero or ad
- sections that do not help someone operate, monitor, or decide

Litmus: if an operator scans only headings, labels, and numbers, can they understand the page immediately?

## Responsive And Accessibility Baseline

- design for both mobile and desktop
- derive styling from semantic tokens
- keep contrast at WCAG 2.1 AA for text and interactive states
- ensure interaction states remain visible
- verify that decorative effects do not reduce legibility

## Optimization Order

When improving an existing UI, prioritize changes in this order:

1. Reconstruct color if the palette is tiring, generic, or cheap-looking.
2. Modernize layout with stronger hierarchy, asymmetry, and spacing.
3. Inject motion and interaction where the page feels static.
4. Upgrade typography and spacing relationships.
5. Refine textures, shadows, borders, and surface details.

## Execution Style

Match implementation complexity to the chosen direction:

- maximalist work can justify richer animation, effects, and denser code
- minimalist work should rely on restraint, precision, spacing, and subtle detail

Do not collapse both into the same average implementation style.

## Anti-Patterns

Do not ship these defaults unless the brand or source material clearly demands them:

- purple gradients on white
- neon color schemes
- evenly distributed rainbow palettes
- safe, repetitive symmetric sections
- generic card grids with no composition idea
- default font stacks with no personality
- placeholder-heavy landing pages
- repeated aesthetic output across unrelated requests

Reject these specific failure combinations:

- generic SaaS card grid as the first impression
- beautiful image with weak brand presence
- strong headline with no clear action
- busy imagery behind text
- sections that repeat the same mood statement
- carousel with no narrative purpose
- app UI made of stacked cards instead of layout
- sections that require many tiny UI devices to explain themselves

Every generation should vary theme, font pairing, and aesthetic treatment enough to avoid a templated feel.

## Litmus Checks

Use these to validate design quality, not just technical correctness:

- Is the brand or product unmistakable in the first screen?
- Is there one strong visual anchor?
- Can the page be understood by scanning headlines only?
- Does each section have one job?
- Are cards actually necessary, or can the content live in plain layout?
- Does motion improve hierarchy or atmosphere, or is it ornamental?
- Would the design still feel premium if all decorative shadows were removed?
