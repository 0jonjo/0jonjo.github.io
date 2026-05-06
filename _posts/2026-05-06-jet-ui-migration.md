---
layout: post
title: "Migrating Rails Views to Jet UI: A Guide with ViewComponent and Tailwind v4"
description: "Ar real-world experience migrating Calcpace to Jet UI, a ViewComponent-based library built on Tailwind CSS v4. How we standardized our UI, leveraged generators, and solved production gotchas."
tags: programming ruby ai english
categories: misc
youtubeId:
image: https://github.com/user-attachments/assets/77774ac0-f804-472e-89ae-670751139237
---

Last week, I wrote an article about migrating Calcpace views to use jet_ui to [JetRockets blog](https://jetrockets.com/blog/migrating-rails-views-to-jet-ui-a-real-world-guide-with-viewcomponent-and-tailwind-v4). Here is the full article:

Maintaining a consistent UI in a growing Rails application is a classic challenge. We often start with the best intentions, clean HTML and utility classes, but as the app scales, we inevitably fall into "UI boilerplate fatigue." Whether it’s copy-pasting the same "Avatar with initials" logic across dozens of views or reinventing the wheel for every animated toast, this duplication slowly erodes our development velocity.

To solve this, I recently migrated [Calcpace](https://calcpace.app), a running and cycling tracker built with Rails 8, to [jet_ui](https://github.com/jetrockets/jet_ui), JetRockets’ component library. In this article, I’ll show you how we used Calcpace as a real-world playground to standardize our interface, leverage Tailwind CSS v4, and solve the production "gotchas" that often come with gem-based assets.

## The Problem: Copy-Paste Debt

Before the migration, our UI was functional but repetitive. Handling profile pictures required manual conditional logic for avatars and initials in every view:

```erb
<%# Before: UI logic leaking into views %>
<% if current_profile.avatar.attached? %>
  <%= image_tag current_profile.avatar.variant(resize_to_fill: [24, 24]), class: "w-6 h-6 rounded-full" %>
<% else %>
  <div class="w-6 h-6 rounded-full bg-gray-200 flex items-center justify-center text-xs">
    <%= current_profile.initials %>
  </div>
<% end %>
```

<img width="615" height="344" alt="image" src="https://github.com/user-attachments/assets/044907e3-1f49-434f-b929-0edc1e33f210" />

This "inline Tailwind" approach lacks a single source of truth. Changing a border radius meant a tedious search-and-replace across the entire codebase.

## The Strategy: Incremental Migration

One of the biggest concerns when adopting a component library is the "big bang" rewrite. Do you have to change every view at once? Absolutely not.

`jet_ui` is designed for incremental adoption. In Calcpace, we didn't touch our legacy views initially. We started by replacing the most "noisy" elements—flashes and avatars—and then moved to complex data tables. You can have a page powered entirely by `jet_ui` components sitting right next to a legacy ERB view using plain Tailwind utility classes. They coexist perfectly because `jet_ui` respects your existing Tailwind configuration while providing the structure of ViewComponent. This removes the psychological barrier of migration: you can improve your app one component at a time.

## Requirements and Plugging in Jet UI

`jet_ui` is built on [ViewComponent](https://viewcomponent.org) and [Tailwind CSS v4](https://tailwindcss.com). It follows a "Rails-native" philosophy, leveraging the latest tools in the ecosystem:

- Ruby >= 3.0 and Rails >= 7.0 (Calcpace runs on Rails 8.1).
- Tailwind CSS v4 (via `tailwindcss-rails >= 4.x`).
- Stimulus and Turbo (standard in modern Rails).

In your `Gemfile`:

```ruby
gem "view_component"
gem "jet_ui"
```

## The Power of Generators

One of `jet_ui`'s standout features is its suite of generators. They don’t just copy files; they wire up your entire application.

### `jet_ui:install`
This sets up the library in your application (CSS + JS). It is safe to re-run after gem upgrades, as already-configured steps are automatically skipped:
```bash
rails generate jet_ui:install
```

### `jet_ui:eject`
If you need to customize a component beyond standard options, you can "eject" it. This copies the Ruby class, ERB template, and Stimulus controller directly into your `app/components/jet_ui/` folder. The ejected files take precedence automatically.

You can eject multiple components at once and use flags to keep your codebase lean:
```bash
# Eject button, card, and flash
rails generate jet_ui:eject btn card flash

# Skip specific files if you only want to customize the template
rails generate jet_ui:eject btn --skip-test --skip-preview
rails generate jet_ui:eject flash --skip-javascript
```

## Production Readiness: The Vendoring Strategy

While the `install` generator works perfectly for local development by pointing to the gem's path, production environments like Docker or CI require a more portable approach. To ensure a deterministic build and clean logs, we adopt a Vendoring strategy. Instead of relying on absolute filesystem paths that change between environments, we copy the CSS directly into the repository but place it outside the standard Rails asset search path to avoid duplicate serving.

1. Vendor the assets programmatically:
   ```bash
   mkdir -p vendor/stylesheets
   cp -r $(bundle show jet_ui)/app/assets/stylesheets/* vendor/stylesheets/
   ```

2. Update your Tailwind source file:
   ```css
   /* app/assets/tailwind/application.css */
   @import "tailwindcss";
   @import "../../../vendor/stylesheets/jet_ui.css";
   ```

Why this approach?
- Portability: The build works in Docker, CI, and any developer's machine without modifications.
- Clean Logs: By placing files in `vendor/stylesheets` (which Propshaft ignores by default), the asset pipeline won't try to serve individual component files (like `popover.css`). This prevents the "404 Not Found" noise in production logs for files already bundled into your main CSS.

<img width="734" height="312" alt="image" src="https://github.com/user-attachments/assets/839cb7b0-a01c-4e3b-9765-2cdf2f5176dc" />

## Customizing the Theme with Tailwind v4

`jet_ui` uses modern CSS variables. Instead of overriding thousands of utility classes, you update the theme variables in your CSS source:

```css
@theme {
  --accent-hue: 163;      /* Calcpace Emerald */
  --accent-chroma: 0.2;
  --accent-lightness: 0.52;
}
```

Every component, from buttons to focus rings, will now use your custom palette.

## Replacing the Noise: Real-World Examples

### Interactive Form Groups
We used `jet_ui.group` to standardize selectors. For our activity unit toggle (KM/MI), the component handles the styling and layout, leaving us with a clean DSL:

```erb
<%= jet_ui.group do %>
  <%= form.radio_button :unit, "km", checked: true %>
  <%= form.radio_button :unit, "mi" %>
<% end %>
```

### Advanced Composition: Tables and Tabs
The World Records page was our "stress test" for displaying dense data. By composing `tabs`, `card`, and `table`, we reduced a complex view to a readable DSL:

```erb
<%= jet_ui.tabs do %>
  <%= jet_ui.tabs_item "KM", href: records_path(unit: "km"), active: @unit == "km" %>
  <%= jet_ui.tabs_item "MI", href: records_path(unit: "mi"), active: @unit == "mi" %>
<% end %>

<%= jet_ui.card class: "overflow-hidden" do %>
  <%= jet_ui.table hovered: true do %>
    <%= jet_ui.table_thead do %>
      <%= jet_ui.table_tr do %>
        <%= jet_ui.table_th { "Event" } %>
        <%# ... %>
      <% end %>
    <% end %>
  <% end %>
<% end %>
```

This composition proves the architectural leverage: we get a professional data grid with integrated navigation, all following the same design system with zero manual CSS.

<img width="1001" height="514" alt="image" src="https://github.com/user-attachments/assets/77774ac0-f804-472e-89ae-670751139237" />

## Why jet_ui? (The Alternatives)

You might ask: "Why not just use Flowbite, shadcn-rails, or RailsUI?"

While those are great tools, they occupy different niches. Flowbite is fantastic for Tailwind-first projects but isn't built as a first-class `ViewComponent` library, often requiring you to wrap their HTML yourself. shadcn-rails follows the "copy-paste" philosophy which is great for total control, but lacks a clean, gem-based upgrade path for those who want their design system managed as a dependency. RailsUI is a premium, template-oriented solution that is excellent for rapid prototyping but might feel too opinionated for existing apps.

`jet_ui` sits in the "Goldilocks" zone: it's ViewComponent-native, Tailwind v4-native, and gem-distributed with an "eject-on-demand" safety valve. You get the maintenance benefits of a gem with the flexibility of local code when you need it.

## Conclusion: Architectural Leverage

Migration isn't just about "fancy" code. It's about reducing cognitive load. Developers can focus on building features using high-level components rather than wrestling with low-level utility classes in every single view. If you're building a modern Rails app, `jet_ui` is the bridge between the flexibility of Tailwind and the structure of a professional design system.

## Links
- 📦 [Jet_UI on RubyGems](https://rubygems.org/gems/jet_ui)
- 📁 [Jet_UI Repo](https://github.com/jetrockets/jet_ui)
- 🏃 [Calcpace](https://calcpace.app)
