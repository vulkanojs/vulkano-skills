---
name: vulkano-frontend-store
description: Use when adding or editing a Pinia store in this Vulkano framework project's frontend/ — store-per-concern splitting, setup-style defineStore, the useAppStore app-shell exception, and store testing.
---

# Frontend Store (Pinia)

## Overview

State worth surviving a re-render lives in Pinia, not local component `ref`/`reactive`: component-local state resets whenever HMR can't hot-swap a module in place and falls back to a full reload, while state in a store is less likely to be lost across that reload. Pinia is installed by default (`app.use(createPinia())` already registered in each entrypoint's `app.js`) — just create the store file, no setup step.

## When to use

- New entity needs shared/fetched state (list, current record, filters)
- Adding app-shell-wide state (loading spinner, socket status, sidebar)
- Editing an existing store's state/actions

Not for a component's own local UI state (a toggle only that component cares about) — plain `ref`/`reactive` in `setup()` is fine for that. Not for the component/view file layout around a store's consumer — see vulkano-frontend-component.

## One store per concern

`store/use<Entity>Store.js`, setup-style (not options-style), no barrel file. If a payload carries data for multiple entities (e.g. an event, its attendee, and a campaign), split it into independent stores rather than one combined store:

```
store/
  useEventStore.js
  useAttendeeStore.js
  useCampaignStore.js
```

Each store owns only its own entity's state, getters, and actions — a component importing `useAttendeeStore` should never need to reach into event or campaign state.

```js
// store/useEventStore.js
import { ref, getCurrentInstance } from 'vue';
import { defineStore } from 'pinia';

export const useEventStore = defineStore('event', () => {
  const { $api } = getCurrentInstance().proxy || {};
  const current = ref(null);

  async function fetch(id) {
    current.value = await $api.get(`/event/${id}`);
  }

  return { current, fetch };
});
```

(setup-style store, in line with the Composition API preference — not the options-style `defineStore('event', { state, actions })`.)

Naming: `use<Entity>Store` (singular, matching the model naming convention), file per store, no aggregator/barrel file — import each store directly where it's used.

## Exception — `useAppStore` for app-shell state

App-shell-level state — things there's only ever one of, shared across the whole app regardless of route — lives in a single `useAppStore`, not split per concern like entity stores: a global loading spinner, Socket.io connection status (`connected`/`reconnecting`/`disconnected`), a sidebar-open flag, a theme toggle. This is the one deliberate exception to "one store per concern": these are all facets of the same app shell, read/written from unrelated places (a router guard, the socket client, any component), so bundling them in one store avoids a proliferation of near-empty singleton stores. Entity data (`useEventStore`, etc.) still stays split. Applies to any area of the app (public site, CMS/admin, widget) that needs this kind of shared state:

```js
// store/useAppStore.js
import { ref } from 'vue';
import { defineStore } from 'pinia';

export const useAppStore = defineStore('app', () => {
  const isLoading = ref(false);
  const socketStatus = ref('disconnected'); // 'connected' | 'reconnecting' | 'disconnected'

  function setLoadingStatus(status) {
    isLoading.value = status;
  }

  function setSocketStatus(status) {
    socketStatus.value = status;
  }

  return { isLoading, socketStatus, setLoadingStatus, setSocketStatus };
});
```

## After writing

- New store → test at `test/frontend/<entrypoint>/store/use<Entity>Store.test.js` (`createPinia()` + `setActivePinia()` in `beforeEach`, mock `$api` at the store boundary — never hit real network). Full pattern table: vulkano-testing skill.
- Run `vp check` and `vp test`.

## Reference

`references/AGENTS/STORE.md` (full detail, same content as this skill).
