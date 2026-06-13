# Her Calendar

> Child Custody Calendar App for co-parents

**⚠️ This project is under development**

## Overview

Her Calendar is a web application designed for co-parents (divorced or separated) to track and visualize child custody schedules. It automatically calculates which parent has custody on any given day based on configurable repeating schedules.

## Features

- **Monthly Calendar View** - Grid-based calendar (Mon-Sun) with month navigation
- **Custody Schedule Tracking** - Automatic calculation based on:
  - Pattern: biweekly or weekly
  - Start/end dates
  - Week rotation direction (which parent starts)
- **Visual Custody Indicators** - Color-coded labels showing which parent has custody
- **Event Management** - Add/delete custom events to specific dates
- **Parent Settings** - Configure two parent names and their colors
- **Schedule History** - View all schedule records and audit trail

## Tech Stack

- Vue 3 + TypeScript + Vite
- Tailwind CSS v4
- @iconify/vue (icons)
- Vue Router
- Pinia (state management)

## Data Models

```typescript
// Custody schedule record
interface ScheduleRecord {
  id: string
  pattern: 'biweekly' | 'weekly'
  startDate: string      // ISO date "YYYY-MM-DD"
  endDate: string | null // null = ongoing
  startWeekWith: 'parent1' | 'parent2'
}

// Custom event
interface CalendarEvent {
  id: number
  title: string
  date: string
  color: string
}

// Parent configuration
interface ParentConfig {
  parent1Name: string
  parent2Name: string
  parent1Color: string
  parent2Color: string
}
```

## Project Status

- [x] Calendar view with navigation
- [x] Custody schedule calculation
- [x] Visual custody indicators
- [x] Event management (add/delete)
- [x] Parent settings
- [x] Schedule history/logs
- [ ] Data persistence (localStorage)
- [ ] Backend integration (optional)

## Development

```sh
# Install dependencies
npm install

# Compile and hot-reload for development
npm run dev

# Type-check, compile and minify for production
npm run build
```

## Recommended IDE Setup

[VS Code](https://code.visualstudio.com/) + [Vue (Official)](https://marketplace.visualstudio.com/items?itemName=Vue.volar) (and disable Vetur).

## Recommended Browser Setup

- Chromium-based browsers (Chrome, Edge, Brave, etc.):
  - [Vue.js devtools](https://chromewebstore.google.com/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd)
  - [Turn on Custom Object Formatter in Chrome DevTools](http://bit.ly/object-formatters)
- Firefox:
  - [Vue.js devtools](https://addons.mozilla.org/en-US/firefox/addon/vue-js-devtools/)
  - [Turn on Custom Object Formatter in Firefox DevTools](https://fxdx.dev/firefox-devtools-custom-object-formatters/)
