## Skeleton

```js
// frontend/<entrypoint>/utils/validators.js
export const V = {
  required: (message) => (value) => (value === '' || value == null ? message : null),
  email: (message) => (value) => (/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) ? null : message)
};
```

```js
// frontend/<entrypoint>/composables/useFormValidation.js
import { reactive } from 'vue';

export function useFormValidation(rules) {
  const fieldErrors = reactive(Object.fromEntries(Object.keys(rules).map((field) => [field, ''])));

  function validate(formData) {
    let isValid = true;

    for (const [field, validators] of Object.entries(rules)) {
      let error = '';

      for (const validator of validators) {
        error = validator(formData[field]) || ''; // validators return null when valid
        if (error) {
          break;
        }
      }

      fieldErrors[field] = error;
      if (error) {
        isValid = false;
      }
    }

    return isValid;
  }

  function clearError(field) {
    fieldErrors[field] = '';
  }

  return { fieldErrors, validate, clearError };
}
```

```js
// Index.js
import { reactive, ref, getCurrentInstance } from 'vue';
import { V } from '@website/utils/validators.js';
import { useFormValidation } from '@website/composables/useFormValidation.js';

export default {
  setup() {
    const { $api } = getCurrentInstance().proxy || {};
    const form = reactive({ email: '', password: '' });
    const isSubmitting = ref(false);
    const flash = ref(null); // { type: 'success' | 'error', message }

    const formRules = {
      email: [V.required('Email is required'), V.email('Enter a valid email address')],
      password: [V.required('Password is required')]
    };

    const { fieldErrors, validate, clearError } = useFormValidation(formRules);

    async function submit() {
      flash.value = null;

      if (!validate(form)) {
        return;
      }

      isSubmitting.value = true;
      try {
        await $api.post('/auth/login', form);
        flash.value = { type: 'success', message: 'Signed in' };
      } catch (err) {
        // Api.js rejects with an Error: message = backend error.detail, status 0 = network
        flash.value = {
          type: 'error',
          message: err.message || 'Could not sign in. Please try again.'
        };
      } finally {
        isSubmitting.value = false;
      }
    }

    return { form, fieldErrors, clearError, isSubmitting, flash, submit };
  }
};
```

```html
<form novalidate @submit.prevent="submit">
  <label class="login__label">
    Email <span class="field-required">*</span>
    <input
      id="email"
      type="email"
      required
      v-model="form.email"
      :class="{ 'login__input--invalid': fieldErrors.email }"
      :aria-invalid="!!fieldErrors.email"
      :aria-describedby="fieldErrors.email ? 'email-error' : null"
      @input="clearError('email')"
    />
  </label>
  <span v-if="fieldErrors.email" id="email-error" class="login__field-error"
    >{{ fieldErrors.email }}</span
  >

  <label class="login__label">
    Password <span class="field-required">*</span>
    <input
      id="password"
      type="password"
      autocomplete="current-password"
      required
      v-model="form.password"
      :class="{ 'login__input--invalid': fieldErrors.password }"
      :aria-invalid="!!fieldErrors.password"
      :aria-describedby="fieldErrors.password ? 'password-error' : null"
      @input="clearError('password')"
    />
  </label>
  <span v-if="fieldErrors.password" id="password-error" class="login__field-error"
    >{{ fieldErrors.password }}</span
  >

  <Transition name="flash">
    <p
      v-if="flash"
      :class="`login__flash login__flash--${flash.type}`"
      :role="flash.type === 'error' ? 'alert' : 'status'"
    >
      {{ flash.message }}
    </p>
  </Transition>

  <button type="submit" :disabled="isSubmitting" :class="{ 'is-loading': isSubmitting }">
    Submit
  </button>
</form>
```

Submit failure (`$api` rejects with an `Error` whose `message` is the backend's `error.detail`, plus `status`) always surfaces as a flash message — never swallow the error silently. Show `err.message`; a custom text only for a specific `err.status` (e.g. `0` → offline). If the project has a UI kit installed (see below), render it with the kit's toast/message component instead of the inline `<p>`.
