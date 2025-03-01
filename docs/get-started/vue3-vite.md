---
title: Storybook for Vue & Vite
---

export const SUPPORTED_RENDERER = 'vue';

Storybook for Vue & Vite is a [framework](../contribute/framework.md) that makes it easy to develop and test UI components in isolation for [Vue](https://vuejs.org/) applications built with [Vite](https://vitejs.dev/). It includes:

- 🏎️ Pre-bundled for performance
- 🪄 Zero config
- 🧠 Comprehensive docgen
- 💫 and more!

<If notRenderer={SUPPORTED_RENDERER}>

<Callout variant="info">

Storybook for Vue & Vite is only supported in [Vue](?renderer=vue) projects.

</Callout>

<!-- End non-supported renderers -->

</If>

<If renderer={SUPPORTED_RENDERER}>

## Requirements

- Vue ≥ 3
- Vite ≥ 3.0 (4.X recommended)
- Storybook ≥ 7.0

## Getting started

### In a project without Storybook

Follow the prompts after running this command in your Vue project's root directory:

<!-- prettier-ignore-start -->

<CodeSnippets
  paths={[
   'common/init-command.npx.js.mdx',
   'common/init-command.yarn.js.mdx',
   'common/init-command.pnpm.js.mdx',
  ]}
/>

<!-- prettier-ignore-end -->

[More on getting started with Storybook.](./install.md)

### In a project with Storybook

This framework is designed to work with Storybook 7+. If you’re not already using v7, upgrade with this command:

<!-- prettier-ignore-start -->

<CodeSnippets
  paths={[
    'common/storybook-upgrade.npm.js.mdx',
    'common/storybook-upgrade.pnpm.js.mdx',
    'common/storybook-upgrade.yarn.js.mdx'
  ]}
/>

<!-- prettier-ignore-end -->

#### Automatic migration

When running the `upgrade` command above, you should get a prompt asking you to migrate to `@storybook/vue3-vite`, which should handle everything for you. In case that auto-migration does not work for your project, refer to the manual migration below.

#### Manual migration

First, install the framework:

<!-- prettier-ignore-start -->

<CodeSnippets
  paths={[
    'vue/vue3-vite-install.npm.js.mdx',
    'vue/vue3-vite-install.pnpm.js.mdx',
    'vue/vue3-vite-install.yarn.js.mdx'
  ]}
/>

<!-- prettier-ignore-end -->

Then, update your `.storybook/main.js|ts` to change the framework property:

<!-- prettier-ignore-start -->

<CodeSnippets
  paths={[
    'vue/vue3-vite-add-framework.js.mdx',
    'vue/vue3-vite-add-framework.ts.mdx'
  ]}
/>

<!-- prettier-ignore-end -->

## Extending the Vue application

Storybook creates a [Vue 3 application](https://vuejs.org/api/application.html#application-api) for your component preview.
When using global custom components (`app.component`), directives (`app.directive`), extensions (`app.use`), or other application methods, you will need to configure those in the `./storybook/preview.js` file.

Therefore, Storybook provides you with a `setup` function exported from this package, which receives as a callback your Storybook instance, which you can interact with and add your custom configuration.

```js
// .storybook/preview.js
import { setup } from '@storybook/vue3';

setup((app) => {
  app.use(MyPlugin);
  app.component('my-component', MyComponent);
  app.mixin({
    /* My mixin */
  });
});

// Rest of the file...
```

## Using `vue-component-meta`

<Callout variant="info">

`vue-component-meta` is only available in Storybook ≥ 8. It is currently opt-in, but will become the default in a future version of Storybook.

</Callout>

[`vue-component-meta`](https://github.com/vuejs/language-tools/tree/master/packages/component-meta) is a tool maintained by the Vue team that extracts metadata from Vue components. Storybook can use it to generate the [controls](../essentials/controls.md) for your stories and documentation. It's a more full-featured alternative to `vue-docgen-api` and is recommended for most projects.

If you want to use `vue-component-meta`, you can configure it in your `.storybook/main.js|ts` file:

```ts
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/vue3-vite';

const config: StorybookConfig = {
  framework: {
    name: '@storybook/vue3-vite',
    options: {
      docgen: 'vue-component-meta',
    },
  },
};

export default config;
```

`vue-component-meta` comes with many benefits and enables more documentation features, such as:

### Support for multiple component types

`vue-component-meta` supports all types of Vue components (including SFC, functional, composition / options API components) from `.vue`, `.ts`, `.tsx`, `.js`, and `.jsx` files.

It also supports both default and named component exports.

### Prop description and JSDoc tag annotations

To provide a description for a prop, including tags, you can use JSDoc comments in your component's props definition:

```html
<!-- YourComponent.vue -->
<script setup lang="ts">
  interface MyComponentProps {
    /** The name of the user */
    name: string;
    /**
     * The category of the component
     *
     * @since 8.0.0
     */
    category?: string;
  }

  withDefaults(defineProps<MyComponentProps>(), {
    category: 'Uncategorized',
  });
</script>
```

The props definition above will generate the following controls:

![Controls generated from props](./vue-component-meta-prop-types-controls.png)

### Events types extraction

To provide a type for an emitted event, you can use TypeScript types (including JSDoc comments) in your component's `defineEmits` call:

```html
<!-- YourComponent.vue -->
<script setup lang="ts">
  type MyChangeEvent = 'change';

  interface MyEvents {
    /** Fired when item is changed */
    (event: MyChangeEvent, item?: Item): void;
    /** Fired when item is deleted */
    (event: 'delete', id: string): void;
    /** Fired when item is upserted into list */
    (e: 'upsert', id: string): void;
  }

  const emit = defineEmits<MyEvents>();
</script>
```

Which will generate the following controls:

![Controls generated from events](./vue-component-meta-event-types-controls.png)

### Slots types extraction

The slot types are automatically extracted from your component definition and displayed in the controls panel.

```html
<!-- YourComponent.vue -->
<template>
  <slot :num="123"></slot>
  <br />
  <slot name="named" str="str"></slot>
  <br />
  <slot name="no-bind"></slot>
  <br />
  <slot name="vbind" v-bind="{ num: 123, str: 'str' }"></slot>
</template>

<script setup lang="ts"></script>
```

If you use `defineSlots`, you can provide a description for each slot using JSDoc comments in your component's slots definition:

```ts
defineSlots<{
  /** Example description for default */
  default(props: { num: number }): any;
  /** Example description for named */
  named(props: { str: string }): any;
  /** Example description for no-bind */
  noBind(props: {}): any;
  /** Example description for vbind */
  vbind(props: { num: number; str: string }): any;
}>();
```

The definition above will generate the following controls:

![Controls generated from slots](./vue-component-meta-slot-types-controls.png)

### Exposed properties and methods

The properties and methods exposed by your component are automatically extracted and displayed in the controls panel.

```html
<!-- YourComponent.vue -->
<script setup lang="ts">
  import { ref } from 'vue';

  const label = ref('Button');
  const count = ref(100);

  defineExpose({
    /** A label string */
    label,
    /** A count number */
    count,
  });
</script>
```

The definition above will generate the following controls:

![Controls generated from exposed properties and methods](./vue-component-meta-exposed-types-controls.png)

### Limitations

`vue-component-meta` cannot currently reference types from an import alias. You will need to replace any aliased imports with relative ones, as in the example below. See [this issue](https://github.com/vuejs/language-tools/issues/3896) for more information.

```ts
// YourComponent.ts
import type { MyProps } from '@/types'; // ❌ Cannot be resolved
import type { MyProps } from '../types'; // ✅ Can be resolved
```

## Troubleshooting

### Storybook doesn't work with my Vue 2 project

[Vue 2 entered End of Life](https://v2.vuejs.org/lts/) (EOL) on December 31st, 2023, and is no longer maintained by the Vue team. As a result, Storybook no longer supports Vue 2. We recommend you upgrade your project to Vue 3, which Storybook fully supports. If that's not an option, you can still use Storybook with Vue 2 by installing the latest version of Storybook 7 with the following command:

<!-- prettier-ignore-start -->

<CodeSnippets
  paths={[
    'common/storybook-init-v7.npx.js.mdx',
    'common/storybook-init-v7.yarn.js.mdx',
    'common/storybook-init-v7.pnpm.js.mdx',
  ]}
/>

<!-- prettier-ignore-end -->

## API

### Options

You can pass an options object for additional configuration if needed:

```ts
// .storybook/main.ts
import type { StorybookConfig } from '@storybook/vue3-vite';

const config: StorybookConfig = {
  framework: {
    name: '@storybook/vue3-vite',
    options: {
      docgen: 'vue-component-meta',
    },
  },
};

export default config;
```

#### `builder`

Type: `Record<string, any>`

Configure options for the [framework's builder](../api/main-config-framework.md#optionsbuilder). For this framework, available options can be found in the [Vite builder docs](../builders/vite.md).

#### `docgen`

Type: `'vue-docgen-api' | 'vue-component-meta'`

Default: `'vue-docgen-api'`

Since: `8.0`

Choose which docgen tool to use when generating controls for your components. See [Using `vue-component-meta`](#using-vue-component-meta) for more information.

<!-- End supported renderers -->

</If>
