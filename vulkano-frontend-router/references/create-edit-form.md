### Create + Edit → one `Form.vue`, not `Create.vue`/`Edit.vue`

Don't split create and edit into two separate views — they're the same form, branching only on whether `:id` is present in the route. One `Form.vue` (routed from both `/product/create` and `/product/edit/:id`) avoids duplicating the whole form markup/validation across two files:

```js
// Form.js
import { ref, onMounted, getCurrentInstance } from 'vue';
import { useRoute, useRouter } from 'vue-router';

export default {
  setup() {
    const { $api } = getCurrentInstance().proxy || {};
    const route = useRoute();
    const router = useRouter();
    const id = route.params.id; // undefined on /product/create
    const form = ref({ name: '', price: 0 });
    const isSubmitting = ref(false);
    const error = ref('');

    async function getProduct(id) {
      try {
        form.value = await $api.get(`/product/${id}`); // prefill for edit
      } catch (err) {
        error.value = err.message;
      }
    }

    onMounted(async () => {
      if (!id) return;
      await getProduct(id);
    });

    async function submit() {
      // validate first — useFormValidation, see vulkano-frontend-form (skipped here)
      isSubmitting.value = true;
      error.value = '';
      try {
        if (id) {
          await $api.put(`/product/${id}`, form.value);
        } else {
          await $api.post('/product', form.value);
        }
        router.push('/product/list');
      } catch (err) {
        error.value = err.message; // backend error.detail
      } finally {
        isSubmitting.value = false;
      }
    }

    return { form, isSubmitting, error, submit };
  }
};
```

Only the `:id` branching is router-specific — fields, validation, flash and loading follow vulkano-frontend-form. Same rule applies to the backend controller (`PUT`/`POST` split on `:id`, not two controllers) — see vulkano-backend-controller.
