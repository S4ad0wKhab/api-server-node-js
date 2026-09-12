# What this is
This is a Node.js API server for Minecraft Earth, a now-discontinued AR game by Microsoft. The server handles authentication, session management, game catalog data (items, recipes, NPCs), and player state persistence. It's a reverse-engineered implementation designed to preserve the game functionality using open-source tooling (Project Genoa).
# Stack
* **Language(s):** TypeScript
* **Framework / runtime:** Express.js + Node.js
* **Notable libraries:** MongoDB (session & player state), nconf (configuration), runtypes (runtime validation)
# How it's organized
```
src/
  app.ts                 Express app setup, catalog loading, DB connection
  config.ts              Configuration via nconf (config.json + env vars)
  db.ts                  MongoDB transaction wrapper with ACID semantics
  
  routes/
    locator.ts           Service environment/CDN discovery endpoint
    signin.ts            Player signin, creates session with token
    resourcepack.ts      Resource pack downloads
    authenticated/       Protected routes (flags, catalogs, player, tappables)
  
  model/
    sessions.ts          Session & request lifecycle management, sequence numbers
    player.ts            Player data model
  
  catalog/
    catalog.ts           Base class for catalog loading from JSON files
    items.ts             Item definitions (Minecraft items & crafting materials)
    recipes.ts           Crafting & smelting recipes
    journal.ts           Game lore & progression entries
    nfc.ts               NFC tag/beacon data
  
  middleware/
    auth.ts              Extracts & validates session token from headers
    log.js               HTTP request logging
    force-content-type-on-304.js  Workaround for broken Minecraft Earth client
  
  utils/
    guid.ts              GUID generation
    queue.ts             Per-session FIFO queue for serializing updates
    api-response-wrapper.ts  Consistent response format

config.json            Server hostname, port, data paths
```
**How it fits together:** On startup, the server loads game catalogs (items, recipes, NPCs) from local JSON files, then connects to a local MongoDB instance. Clients discover the service via /player/environment, then post a signin ticket to /api/v1.1/player/profile/signin to get a session token. Authenticated requests include the token in an Authorization: Genoa <token> header and a Session-Id header. Per-session FIFO queues prevent race conditions on player state updates. Sequence numbers track changes to profile, inventory, challenges, and other mutable sections, allowing delta sync to clients.
# How to run it
```
npm install
npm run build    # Compiles TypeScript to build/
npm start        # Runs build/app.js (not defined in package.json; use node build/app.js)
```
You'll need:
* **Node.js 18+** (uses crypto.randomBytes)
