# Detect empty GrapesJS pages with Vue 3

This example showing how to detect if the currently active page in GrapesJS editor is empty or not.

The main check is in `isPageEmpty(page: Page)` function:

```typescript
function isPageEmpty(page: Page) {
  return page.getMainComponent().components().length === 0
}
```

Because this check works with the list of actual components that GrapesJS understands and provides we can also check if the page has just one very specific component:

```typescript
function isPageSomewhatSpecial(page: Page) {
  if (page.getMainComponent().components().length > 0) {
    const component = page.getMainComponent().components().at(0)

    return component?.getInnerHTML()?.includes('not') || false
  } else {
    return false
  }
  return page.getMainComponent().components().length === 0
}
```
