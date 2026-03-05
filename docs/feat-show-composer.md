# feat: Show Composer in Tracklist & Playlist Views

## Summary

This change adds composer display to all music tracklist and playlist views in Jellyfin Web.
The Jellyfin API only returns fields that are explicitly requested. `People` (which includes composers, conductors, etc.) was missing from several API queries, and the list renderer also needed to be told to display composer information.

---

## Root Cause

Two things had to be in place for composers to appear:

1. **Fetch layer** – `People` must be included in the `Fields` parameter of the API query, otherwise the server simply does not return the data.
2. **Render layer** – The list view component must receive `composer: true`, otherwise the data is ignored even if it is present.

---

## Changed Files

### 1. `src/scripts/playlistViewer.js`
**Layer:** Fetch  
Added `People` to the `Fields` query parameter in `getFetchPlaylistItemsFn`.  
Added `composer: true` to the `listView.getListViewHtml` call in `getItemsHtmlFn`.

```js
// Before
Fields: 'PrimaryImageAspectRatio,MediaSourceCount,Chapters,Trickplay'

// After
Fields: 'PrimaryImageAspectRatio,MediaSourceCount,Chapters,Trickplay,People'
```

```js
// Added to listView.getListViewHtml options
composer: true
```

> **Note:** `playlistViewer.js` is loaded via dynamic `import()` in `itemDetails/index.js`,
> so Webpack bundles it into its own separate chunk (lazy loading).

---

### 2. `src/controllers/itemDetails/index.js`
**Layer:** Fetch  
Added `People` to the `Fields` query for album tracklist items.

---

### 3. `src/components/listview/listview.js`
**Layer:** Render  
Added handling for the `composer` option so the list view can render composer information when present.

---

### 4. `src/components/indicators/useTextLines.tsx`
**Layer:** Render  
Added composer to the text line logic that determines which metadata lines are shown per item.

---

### 5. `src/types/listview/types.ts`
**Layer:** Types  
Added `composer` to the options type definition for the list view.

---

### 6. `src/components/listview/ListItemBody.tsx`
**Layer:** Render  
Added the actual rendering of the composer name in the list item body.

---

### 7. `src/components/ItemsView/ItemsView.tsx` + `items.ts`
**Layer:** Fetch + Render (Experimental UI)  
Added `People` to the API query and `composer: true` to the render options for the experimental items view.

---

## Architecture Note

There is no central data-fetching layer in Jellyfin Web – each view manages its own API query independently:

| View | File | 
|------|------|
| Album tracklist | `itemDetails/index.js` |
| Songs tab | `songs.js` |
| Playlist viewer | `playlistViewer.js` |
| Experimental UI | `ItemsView.tsx` / `items.ts` |

This means any new view that needs to display composers must explicitly include `People` in its `Fields` query **and** pass `composer: true` to the list renderer.
