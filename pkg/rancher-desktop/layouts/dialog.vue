<!--
  - This layout is used by dialog boxes that do not want navigation.
  - With the help of .../pkg/rancher-desktop/window/index.ts, this also handles automatic sizing
  - of the dialog boxes, using the given events:
  - emit    'dialog/load':     the dialog has been mounted.
  - receive 'dialog/populate': any additional data for the dialog.
  - emit    'dialog/ready':    the dialog is ready to be displayed.
  - The page component may set "data-flex" on the root element to a whitespace-
  - separated list of "width" or "height" to help implement flexbox behaviour;
  - however, this interacts badly with automatic resizing, and should only be
  - used for larger dialogs.
  -->

<template>
  <div ref="wrapper" class="wrapper" open>
    <Nuxt class="body" />
  </div>
</template>

<script lang="ts">
import { ipcRenderer } from '@pkg/utils/ipcRenderer';
import Vue from 'vue';

export default Vue.extend({
  head() {
    // If dark-mode is set to auto (follow system-prefs) this is all we need
    // In a possible future with a three-way pref
    // (Always off // Always on // Follow system pref)
    // the "dark" part will be a dynamic pref.
    // See https://github.com/rancher/dashboard/blob/3454590ff6a825f7e739356069576fbae4afaebc/layouts/default.vue#L227 for an example
    return { bodyAttrs: { class: 'theme-dark' } };
  },
  mounted() {
    // The page component is mounted before the layout (because the layout
    // contains the page component); so we can safely send `dialog/load` here
    // and assume the page has already been mounted.
    ipcRenderer.on('dialog/populate', async() => {
      await this.$nextTick();
      (this.$refs.wrapper as Element)?.setAttribute('data-loaded', '');
      ipcRenderer.send('dialog/ready');
    });
    ipcRenderer.send('dialog/load');
  },
});
</script>

<style lang="scss">
  html {
    height: initial;
  }
  body {
    overflow: hidden;
  }
</style>

<style lang="scss" scoped>
@import "@pkg/assets/styles/app.scss";

.wrapper {
  background-color: var(--body-bg);
  border: none;
  color: var(--body-text);
  min-width: 24rem;
  padding: 1.25rem;
  margin: 0 auto;
  display: flex;
  flex-flow: column;
}

.body {
  display: flex;
  flex-flow: column;
  flex-grow: 1;
  gap: 1rem;
}

</style>
