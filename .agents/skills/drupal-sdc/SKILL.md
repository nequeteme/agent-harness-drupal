---
name: drupal-sdc
description: Create and maintain Drupal Single Directory Components (SDC) -- directory structure, component.yml schema, Twig templates, props/slots, compound patterns, and review criteria.
metadata:
  source: drupal.org/project/ai_agents_experimental_collection
  source_license: GPL-2.0-or-later
  original_agent: sdc_generator
version: 1.0.0
---

# Drupal Single Directory Components (SDC)

You are an expert at creating Single Directory Components for Drupal 10.3+ themes. SDC is Drupal core's standard for self-contained, reusable UI components.

## SDC Directory Structure

Each component lives in its own directory under the theme's `components/` folder, organized by atomic design categories:

- `01-atoms/` -- Basic building blocks (buttons, badges, icons)
- `02-molecules/` -- Combinations of atoms (cards, form fields, media objects)
- `03-organisms/` -- Complex sections (headers, footers, hero banners, card grids)

Each component directory contains:

- `{name}.component.yml` -- Metadata and props schema (REQUIRED)
- `{name}.twig` -- The Twig template (REQUIRED)
- `{name}.css` -- Component-scoped styles (optional but recommended)
- `{name}.js` -- JavaScript behaviors (only if interactive)

## Component YAML Format

The `.component.yml` file defines the component's interface:

```yaml
$schema: https://git.drupalcode.org/project/drupal/-/raw/HEAD/core/assets/schemas/v1/metadata.schema.json
name: Human Readable Name
status: stable
subtype: card
group: Molecules/Marketing
description: 'A short description of what this component does and when to use it.'
long_description: |
  A longer description with more detail.
visual_description: |
  What the component looks like visually.
typical_usage: |
  Where and how this component is typically used.
props:
  type: object
  properties:
    title:
      type: string
      title: Title
      examples: ["Example Title"]
    description:
      type: string
      title: Description
      examples: ["Example description text"]
    variant:
      type: string
      title: Variant
      enum: ["primary", "secondary", "outline"]
      default: "primary"
    full_width:
      type: boolean
      title: Full Width
      default: false
```

## Prop Types

SDC supports standard JSON Schema prop types:

**Simple types:**

- `type: string` -- text values, also use for links/URLs
- `type: boolean` -- true/false toggles
- `type: integer` -- whole numbers
- `type: number` -- decimal numbers

**Object types (structured data):**

```yaml
image:
  type: object
  title: Image
  properties:
    src:
      type: string
      title: Image source URL
    alt:
      type: string
      title: Alt text
    width:
      type: integer
      title: Width
    height:
      type: integer
      title: Height
  examples:
    - src: 'https://placehold.co/1920x1080'
      alt: 'Placeholder image'
      width: 1920
      height: 1080
```

**Array types (lists):**

```yaml
tags:
  type: array
  title: Tags
  items:
    type: string
  examples:
    - ["Drupal", "Frontend", "SDC"]
```

**Complex structured props (buttons, links, CTAs):**

For buttons, links, and similar structured items, you can model them as objects or flatten them into separate string props depending on the use case:

```yaml
# Option A: Object prop (clean, grouped)
primary_button:
  type: object
  title: Primary Button
  properties:
    label:
      type: string
      title: Label
    url:
      type: string
      title: URL
  examples:
    - label: "Get Started"
      url: "https://example.com"

# Option B: Flat string props (simpler, more explicit)
primary_button_label:
  type: string
  title: Primary Button Label
  examples: ["Get Started"]
primary_button_url:
  type: string
  title: Primary Button URL
  examples: ["https://example.com"]
```

## Props Are the Default

**ALWAYS use props for all component content.** Every piece of data the component displays -- titles, descriptions, buttons, links, images -- should be a prop. Props give the site builder direct, field-level control over each value.

**Example -- Hero component with props:**

```yaml
props:
  type: object
  properties:
    media:
      type: object
      title: Background image
      properties:
        src:
          type: string
        alt:
          type: string
        width:
          type: integer
        height:
          type: integer
      examples:
        - src: 'https://placehold.co/1920x1080'
          alt: 'Hero background'
          width: 1920
          height: 1080
    heading:
      type: string
      title: Heading
      examples: ["Welcome to Our Site"]
    subheading:
      type: string
      title: Subheading
      examples: ["Discover what we have to offer"]
    primary_button_label:
      type: string
      title: Primary Button Label
      examples: ["Get Started"]
    primary_button_url:
      type: string
      title: Primary Button URL
      examples: ["https://example.com"]
    height:
      type: string
      title: Height
      enum: ['full', 'large', 'ribbon']
      default: 'full'
```

## Slots -- Only When Explicitly Requested

Slots are for composing other components inside your component. **Do NOT use slots by default.** Only use slots when:

1. The user explicitly asks for composability -- keywords like "insert", "embed", "nest", "drop in", "place another component inside", "composable", "slot"
2. You are creating a **container component** for the Compound Component Pattern (see below)

**When in doubt, use props.** A hero with heading + description + button should use props, NOT a slot.

**YAML syntax for slots:**

```yaml
slots:
  items:
    title: "Content items"
```

**Twig syntax -- use `{% block %}` for each slot:**

```twig
<div class="grid">
  {% block items %}{% endblock %}
</div>
```

## Twig Template Conventions

- The Twig template MUST use ALL props and slots defined in component.yml -- every prop must be referenced, every slot must have a `{% block slot_name %}{% endblock %}`
- Props from the YAML are available directly as Twig variables
- For image object props, access sub-properties: `{{ media.src }}`, `{{ media.alt }}`, `{{ media.width }}`, `{{ media.height }}`
- Render images with standard `<img>` tags or Drupal's responsive image patterns:
  ```twig
  {% if media.src %}
    <img src="{{ media.src }}" alt="{{ media.alt|default('') }}"
         width="{{ media.width }}" height="{{ media.height }}"
         loading="lazy">
  {% endif %}
  ```
- Use conditional rendering with `{% if prop_name %}` for optional props
- Use `{{ prop_name|default('fallback') }}` for safe defaults
- Keep templates semantic with proper HTML elements
- Ensure accessibility: proper heading levels, alt text, ARIA attributes
- Follow the existing theme patterns (check existing components for reference)

## CSS and Styling

Two main approaches, depending on the theme's setup:

**Tailwind CSS (if the theme uses it):**

Use utility classes directly in Twig templates. No separate CSS file needed for most components.

```twig
<section class="py-16 px-4 bg-gray-50">
  <h2 class="text-3xl font-bold text-center mb-8">{{ heading }}</h2>
</section>
```

**Component-scoped CSS:**

Write styles in the component's `.css` file. Use the component name as a namespace:

```css
.hero-banner {
  position: relative;
  overflow: hidden;
}
.hero-banner__heading {
  font-size: 2.5rem;
  font-weight: 700;
}
```

## Compound Component Pattern (Container + Item)

This pattern applies when the user asks for a component that displays **multiple repeated complex items** -- e.g. "a grid of cards", "a testimonial carousel", "a pricing table with tiers", "a team members section".

When the component contains repeated items that are complex (more than one field per item), create TWO components:

1. **Container component** (organism) -- provides the layout grid/wrapper and an `items` slot
2. **Item component** (molecule) -- renders a single item with its own props

**Example -- Card Grid (container) + Blog Card (item):**

Container (`card-grid.component.yml`):

```yaml
name: Card Grid
group: Organisms/Layout
props:
  type: object
  properties:
    columns:
      type: string
      title: Columns
      enum: ['2', '3', '4']
      default: '3'
slots:
  items:
    title: "Card items"
```

Container Twig (`card-grid.twig`):

```twig
<div class="grid grid-cols-1 md:grid-cols-{{ columns }}">
  {% block items %}{% endblock %}
</div>
```

Item (`card-blog.component.yml`):

```yaml
name: Blog Card
group: Molecules/Content
props:
  type: object
  properties:
    title:
      type: string
      title: Title
      examples: ["Blog Post Title"]
    excerpt:
      type: string
      title: Excerpt
      examples: ["A short summary of the blog post."]
    image:
      type: object
      title: Card Image
      properties:
        src:
          type: string
        alt:
          type: string
        width:
          type: integer
        height:
          type: integer
      examples:
        - src: 'https://placehold.co/400x300'
          alt: 'Blog image'
          width: 400
          height: 300
```

**When to use this pattern:**

- Feature grids, card grids, testimonial sliders, pricing tables, team member sections, FAQ accordions, timeline entries -- anything with repeated structured content.

**When NOT to use this pattern:**

- Simple lists of strings (tags, breadcrumbs) -- use an array prop with `items: { type: string }`.
- Single-instance content -- just use props and/or a single slot.

## External Libraries via CDNJS

When a component needs a third-party library (slider, lightbox, chart, animation, etc.), include them via `libraryOverrides` in the component.yml.

```yaml
libraryOverrides:
  js:
    https://cdnjs.cloudflare.com/ajax/libs/Swiper/11.0.5/swiper-bundle.min.js: { }
  css:
    component:
      https://cdnjs.cloudflare.com/ajax/libs/Swiper/11.0.5/swiper-bundle.min.css: { }
```

**Key rules:**

- Always use the minified version (`.min.js` / `.min.css`)
- JS goes under `libraryOverrides.js` with the full CDN URL as the key and `{ }` as the value
- CSS goes under `libraryOverrides.css.component` with the full CDN URL as the key and `{ }` as the value
- If the component also has a local JS file, include both:
  ```yaml
  libraryOverrides:
    js:
      https://cdnjs.cloudflare.com/ajax/libs/Swiper/11.0.5/swiper-bundle.min.js: { }
      slider.js: { attributes: { type: module } }
    css:
      component:
        https://cdnjs.cloudflare.com/ajax/libs/Swiper/11.0.5/swiper-bundle.min.css: { }
  ```

## Naming Conventions

- Component machine names: lowercase with hyphens (e.g., `hero-banner`, `card-product`, `pricing-table`)
- Category selection:
  - Atoms: Single-purpose elements (button, badge, icon, avatar)
  - Molecules: Small combinations (card, form-field, stat-item, testimonial)
  - Organisms: Full sections (hero, header, footer, feature-grid, pricing-section)

## Workflow: Creating a New Component

1. **Check for existing components** -- look in `components/` for similar names/descriptions. Avoid duplicates.
2. **Create the directory** under the appropriate atomic design category
3. **Write the component.yml** with full metadata, description, and props schema. Include `examples` for every prop.
4. **Write the Twig template** using the props defined in the YAML
5. **Write the CSS** if any custom styles are needed beyond Tailwind (component-scoped)
6. **Write the JS** only if the component needs interactive behavior
7. **Clear Drupal cache** (`drush cr`)
8. **Verify the component** renders without errors
9. **If the component has repeated complex items**, create the item component too (see Compound Component Pattern). Create the item first, then the container.

## Workflow: Editing an Existing Component

1. **Identify the component** by finding it in the `components/` directory
2. **Make changes** to the relevant files (yml, twig, css, js)
3. **Clear Drupal cache** (`drush cr`)
4. **Verify the component** still renders after your changes

## Review Criteria

When reviewing SDC components, evaluate on these criteria (rate 1-10 each):

1. **Visual Quality** -- layout, spacing, typography, colors
2. **Accessibility** -- semantic HTML, contrast, readability, ARIA attributes, heading hierarchy
3. **Responsiveness** -- works at mobile (375px), tablet (768px), and desktop (1280px) viewports
4. **Completeness** -- all expected content renders, all props are used in the template
5. **JavaScript Health** -- no console errors, no missing resources
6. **Interactivity** -- interactive elements work as expected (if applicable)

**Common issues to check for:**

- Broken layout (elements overlapping or misaligned)
- Missing content (props defined but not rendered)
- Poor contrast (text hard to read against background)
- Missing spacing (elements too cramped or too spread out)
- Non-semantic HTML (divs where headings/lists/nav should be)
- Accessibility gaps (missing alt text, poor heading hierarchy, no focus styles)
- Unscoped CSS (styles leaking out of the component)
- Unused props (defined in YAML but not used in Twig)
- JavaScript errors in the browser console
- Missing CDN libraries (404 network errors)
- Broken interactive behavior (click handlers not working, animations stuck)
- Layout breakage at mobile viewports
- Twig syntax errors or undefined variables

## Tips

- Always include `examples` in the YAML props -- they are used for preview rendering and documentation
- For image examples, use `https://placehold.co/{width}x{height}` as the src
- Use meaningful default values for enum and boolean props
- Make components responsive (mobile-first with breakpoint prefixes)
- Keep Twig templates clean and readable
- Use descriptive `group` values in YAML (e.g., "Molecules/Marketing", "Organisms/Layout")

## Example Prompts

**Creating components:**

- "Create a hero banner component with a title, subtitle, background image, and CTA button"
- "Build a testimonial card with author name, avatar, quote text, and star rating"
- "Create a pricing table with plan name, price, feature list, and signup button"
- "Build a footer with logo, navigation links, and social media icons"
- "Create a feature grid with 3 columns, each having an icon, title, and description"
- "Build an accordion FAQ component with collapsible items using vanilla JavaScript"

**Complex components:**

- "Create a complete navigation bar with logo, menu items, search toggle, and mobile hamburger menu"
- "Create a tabbed content component with accessible keyboard navigation"
- "Create an image gallery component with lightbox support using the GLightbox CDN library"

## Related Skills

- **drupal-frontend-expert** -- Broader frontend/theming expertise, Twig templates, libraries, Drupal.behaviors
- **twig-templating** -- Twig syntax and Drupal-specific extensions
- **drupal-coding-standards** -- Code style and quality standards
