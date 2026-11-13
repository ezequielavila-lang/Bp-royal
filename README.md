# BP Royal

BP Royal is a browser game built with Three.js, TypeScript, and CSS. Vite builds the browser app into `dist/`; the Node server hosts that build and the multiplayer room WebSocket endpoint.

## Requirements

- Node.js 20.19 or newer
- npm

## Develop locally

```sh
npm ci
npm run dev
```

Open <http://localhost:5173/fnbrcopy.html>. Vite reloads the game as files change and forwards room WebSocket traffic to the local Node room server. The room server listens on port 8000.

## Build and run the production version

```sh
npm run build
npm start
```

Open <http://localhost:8000/fnbrcopy.html>. The same Node process serves the built frontend and multiplayer rooms. `Open-fnbrcopy.bat` performs the install/build steps and opens this local URL on Windows.

The release build minifies JavaScript and CSS, omits source maps, hashes output files for long-term caching, and writes a Vite asset manifest. Three.js is split into its own runtime chunk so gameplay updates can reuse the cached engine file. All files under `public/assets/` are staged into `dist/`.

## Rendering performance

Press **F3** in the game to show the live FPS, draw calls, triangle count, and render scale. The renderer lowers pixel resolution when FPS stays below its target and raises it gradually when the machine has spare headroom. Repeated trees use instanced meshes, distant forest trees skip shadows, and the sun shadow camera follows the player with a smaller shadow map. The HUD is for comparing runs on your device; actual FPS depends on its GPU and browser.

## Source layout

- `fnbrcopy.html` contains the game interface markup and keeps the existing `/fnbrcopy.html` entry URL.
- `src/style.css` contains the interface styling.
- `src/main.ts` contains the existing Three.js game, now compiled and bundled through Vite. The old gameplay entry still has shared state while it is being converted; bot combat movement and room-server URL selection are isolated in strictly typed modules under `src/game/` and `src/network/`.
- `server.js` handles room creation, room membership, and multiplayer state messages.
- `public/assets/` contains static images copied into the production build.

The browser owns rendering and the current match simulation. The Node server handles room connections and shares player state; each player's browser still simulates its own bots, loot, building, and combat. Moving those systems to Python or C++ would require a server-authoritative simulation protocol and is a separate gameplay/networking project. Browser game logic runs client-side, so TypeScript is the simplest place to improve and profile it first. Native code can be added later as WebAssembly if profiling identifies a CPU-heavy system that needs it.

## Deploy from GitHub to Render

The included `render.yaml` configures a single Node web service that builds the frontend and serves both the game and WebSocket rooms. This matches the current in-memory room design and avoids a separate database or socket service.

1. Push the project to a GitHub repository.
2. In Render, choose **New > Blueprint**, connect the repository, and apply `render.yaml`.
3. After the service deploys, open `https://<your-service>.onrender.com/fnbrcopy.html` and share that address.

The room server currently stores rooms in process memory, so keep one service instance. If you later add multiple server instances, move room state and message fan-out into shared storage first.

Vercel can host the static Vite build, and its Functions now support WebSockets in public beta. This game's room map is process-local, though, while Vercel connections can land on different function instances and have a maximum connection duration. A Vercel deployment would need shared room storage and reconnect/resync handling; the included Render setup keeps the current multiplayer behavior simpler.

If you host the frontend and room server on different domains, set `VITE_ROOM_SERVER_URL` to the room server's HTTPS URL at build time. The client switches it to secure WebSockets automatically. The included single-host Render setup leaves this unset.
