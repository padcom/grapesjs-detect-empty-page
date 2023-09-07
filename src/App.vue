<template>
  <h1 class="text-3xl text-center">Is page empty? <code>{{ isSelectedPageEmpty }}</code></h1>
  <h1 class="text-3xl text-center">Is page special? <code>{{ isSelectedPageSpecial }}</code></h1>

  <div>
    <label>Select page</label>
    <select :value="selectedPage?.id" @input="selectPage">
      <option v-for="page in pages" :key="page.id" :value="page.id">
        {{ page.id }}
      </option>
    </select>
  </div>

  <div ref="gjs" />
</template>

<script lang="ts" setup>
import { ref, shallowRef, onMounted, computed } from 'vue'
import grapesjs, { Editor, Page } from 'grapesjs'

// Grab a hold on to the editor instance so we can use it everywhere
const editor = shallowRef<Editor>()

// This is the list of all available pages. It is updated when the editor
// is loaded and when pages are being added/removed
const pages = shallowRef([] as Page[])

// This is a reference to the currently selected page. It is updated when
// the page is loaded, pages are added/removed or when the currently selected
// page has been changed
const selectedPage = shallowRef<Page>()

// This checks if the current page has any components added to it
function isPageEmpty(page: Page) {
  return page.getMainComponent().components().length === 0
}

// This is a computed property that returns true/false depending on
// if the currently selected page is empty
const isSelectedPageEmpty = computed(() => selectedPage.value ? isPageEmpty(selectedPage.value) : true)

// We can also interrogate the content to see if there is a component
// that has some special content or properties. Sky is the limit!
const isSelectedPageSpecial = computed(() => {
  const component = selectedPage.value?.getMainComponent()?.components()?.at(0)

  return component?.getInnerHTML()?.includes('not') || false
})

// Simply select the given page. Used here because TypeScript doesn't know
// about the `value` of `target` - so we're doing an `any` here instead
function selectPage(e: any) {
  editor.value?.Pages.select(e.target.value)
}

// This plugin wraps updates to the list of pages and to the currently selected page
function pagesPlugin(ed: Editor) {
  function update() {
    pages.value = ed.Pages.getAll()
    selectedPage.value = ed.Pages.getSelected()
  }

  ed.on('load', update)
  ed.on('page:add', update)
  ed.on('page:remove', update)
  ed.on('page:select', (p: Page) => {
    selectedPage.value = p
    console.log(selectedPage.value?.getMainComponent().components().at(0))
  })
}

// Reference to the root element where GrapesJS is mounted
const gjs = ref<HTMLElement>()

onMounted(() => {
  // Once the component is mounted let's initialize GrapesJS
  editor.value = grapesjs.init({
    // We already have a reference to the HTML element where
    // the editor should be mounted - let's use it instead of CSS selector
    container: gjs.value,
    // We don't want the page to be persisted
    storageManager: false,
    plugins: [
      pagesPlugin,
    ],
    pageManager: {
      // For convenience let's create 2 pages, an empty one and one with some content
      pages: [
        { id: 'empty' },
        { id: 'non-empty', component: '<h1>This page is not empty</h1' },
      ],
    },
  })
})
</script>

<style lang="postcss">
@import "grapesjs/dist/css/grapes.min.css";

.gjs-editor {
  /*
    Since we're using TailwindCSS and it resets some styles we need to restore
    the line-height for GrapesJS or else it looks ugly as hell
  */

  line-height: initial;
}

code {
  @apply font-bold;
}
</style>
