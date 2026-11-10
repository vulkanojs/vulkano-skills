## List view — 4 states

Every list (or any fetched view) renders loading, error, empty and data — never a blank area. `getAll` resolves to a page object: read `.items` (+ `page`, `totalPages`…); a bare array only when the request sent `?page=all`.

```js
// views/Product/List.js
import { ref, onMounted, getCurrentInstance } from 'vue';

export default {
  setup() {
    const { $api } = getCurrentInstance().proxy || {};
    const items = ref([]);
    const isLoading = ref(false);
    const error = ref('');

    async function load() {
      isLoading.value = true;
      error.value = '';
      try {
        items.value = (await $api.get('/product')).items ?? []; // Api.js resolves {} when data is empty
      } catch (err) {
        error.value = err.message; // Api.js: backend error.detail, status 0 = network
      } finally {
        isLoading.value = false;
      }
    }

    onMounted(load);

    return { items, isLoading, error, load };
  }
};
```

<!-- prettier-ignore -->
```html
<!-- views/Product/List.vue -->
<p v-if="isLoading" class="products__loading">Loading…</p>
<div v-else-if="error" class="products__error" role="alert">
  {{ error }} <button type="button" @click="load">Retry</button>
</div>
<p v-else-if="!items.length" class="products__empty">No products yet.</p>
<ul v-else class="products__list">…</ul>
```

Data shared with other views → the same states, with `items`/`isLoading`/`error` and `load` living in the store (vulkano-frontend-store).
