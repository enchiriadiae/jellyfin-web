# Deployment Workflow (manual, pre-merge)

Until the changes are merged into `main` and available via the official Jellyfin Docker image,
the built web frontend needs to be deployed manually to the container.

## Prerequisites
- Dev-Machine (i.e. a Mac): jellyfin-web repo checked out, Node.js + npm installed
- Server Machine (i.e. called Tuxi): Docker running, container named `jellyfin`
- SFTP/SCP access from Mac to Tuxi (`/tmp/dist`)

---

## Steps

### 1. Build on Mac
```bash
cd ~/Documents/Dev/jellyfin/jellyfin-web
rm -rf node_modules/.cache
npm run build:production
```

### 2. Verify the build
Check that the relevant chunk contains the expected changes:
```bash
grep -ro "PrimaryImageAspectRatio,MediaSourceCount,Chapters[^\"']*" dist/39469.*.chunk.js
# Expected: PrimaryImageAspectRatio,MediaSourceCount,Chapters,Trickplay,People
```

### 3. Copy dist to Tuxi
Via SFTP/SCP – copy the entire `dist/` folder to `/tmp/dist` on Tuxi.

### 4. Copy into Docker container
```bash
docker cp /tmp/dist/. jellyfin:/usr/share/jellyfin/web/
```

### 5. Clean up old chunks (important!)
Old chunks with the same chunk number but different hash will shadow the new ones.
Always check and remove them:
```bash
docker exec jellyfin ls /usr/share/jellyfin/web/ | grep "39469"
# If more than one entry: delete the old hash
docker exec jellyfin rm /usr/share/jellyfin/web/<old-chunk-filename>.js
docker exec jellyfin rm /usr/share/jellyfin/web/<old-chunk-filename>.js.LICENSE.txt
```

### 6. Hard-refresh in browser
`Cmd+Shift+R` (Mac) or `Ctrl+Shift+R` (Windows/Linux) to bypass browser cache.

---

## Pitfalls

| Problem | Cause | Fix |
|---------|-------|-----|
| Changes not visible | Old chunk still in container | Step 5 – delete old chunk |
| Changes not in build | Webpack cache | `rm -rf node_modules/.cache` before build |
| `People` missing in API response | `Fields` param incomplete | Check `playlistViewer.js` and `index.js` |
| Composers not rendered | `composer: true` missing | Check `listview.js` call |
