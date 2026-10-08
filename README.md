# Circuit Studio

A modular, browser-first electronics design workspace prototype. It runs without third-party packages and keeps projects in the browser. A small Node.js backend serves the app, provides local catalog/assistant endpoints, and can optionally query attributed public KiCad symbol metadata. No API key, account, cloud database, simulator, or hardware toolchain is required for the default experience.

## Run locally

Requires Node.js 20+ (built-in `fetch` and `AbortSignal.timeout` are used). From the project root:

```sh
npm run dev
```

Open `http://localhost:4173`. `npm start` runs the same server. Run the current test suite with `npm test`.

Projects autosave to `localStorage`; use **Save version** / `Ctrl+S` for a local snapshot. **Share** creates a portable URL containing a design snapshot. It is not a hosted collaboration session.

## Workspace features

- Interactive SVG schematic with orthogonal nets, pin selection, grid snapping, selection, move, pan, zoom, wire tool, properties, copy/paste, duplicate, undo/redo, and keyboard shortcuts.
- Searchable offline starter catalog with 70 records across controller boards, SBCs, sensors, displays, communications, power, passives, semiconductors, and connectors.
- ESP32 room-monitor starter with DHT22, SSD1306 I²C display, 4.7 kΩ data pull-up, series-limited status LED, and 100 nF local bypass capacitor.
- Local design templates for room monitoring, LED control, and irrigation; backend catalog recommendations; optional public symbol-metadata lookup; design cautions; connection summaries; and an editable Arduino/ESP32 firmware scaffold.
- Local behavioral demonstration console, illustrative waveform viewer, and limited structural checks. Values are example samples, not SPICE or hardware measurements.
- PCB-placement preview, manual footprint movement, BOM CSV, schematic SVG, project JSON, KiCad PCB placement skeleton, and draft Gerber/Excellon text exports.
- Local dark/light themes, browser project persistence, and responsive panels.

## Backend and integration routes

The **Backend & integrations** panel shows current API capability and source links. API routes are served by the same Node process:

| Route | Purpose |
| --- | --- |
| `GET /api/health` | Backend, catalog, and optional-model status |
| `GET /api/integrations` | Capability and source metadata |
| `GET /api/catalog?q=...` | Search the starter catalog |
| `GET /api/catalog/open?q=...` | Query the official public KiCad symbol-library metadata when internet access is available |
| `POST /api/assistant` | Local catalog review and optional, user-enabled server-model review |
| `POST /api/ai/test` | Send a fixed, project-free prompt to test the configured model connection |
| `POST /api/import/tinkercad-bom` | Normalize a user-exported Tinkercad component-list CSV and suggest heuristic matches |

### Public component data

The server searches KiCad's official public `kicad-symbols` GitLab repository for matching symbol metadata such as library name, symbol description, footprint hint, keywords, and linked datasheet. Responses include source links and the KiCad library license reference (**CC-BY-SA 4.0**). Results are for discovery only; they do not validate a pinout, package, electrical rating, or design. They are not silently copied into the offline catalog. Internet access is needed for this optional search; an in-memory cache is used by the running server.

### Tinkercad Circuits

Use Autodesk Tinkercad's own export flow to export its component-list CSV, then open **Backend & integrations → Choose CSV**. Circuit Studio previews normalized rows and suggested matches; optionally, matched rows can be added as **unwired** schematic components. The CSV does not contain/reconstruct all circuit connectivity in this bridge. No Tinkercad login automation, private endpoint, or general project-generation API is used. See the [Tinkercad export guide](https://www.tinkercad.com/blog/tinkertips-export-options).

### Cirkit Designer

The integration panel links to public Cirkit Designer documentation. The documented simulation API relates to custom simulation logic inside Cirkit; no external Cirkit project-generation, import/export, or user-account API has been verified or connected. This build does not claim a live project integration. See [Cirkit custom simulation API docs](https://learn.cirkitdesigner.com/custom-simulation-parts/component-simulation-logic-ts/overview/).

### Optional server-side AI

By default, assistant behavior uses local deterministic templates and the backend's local catalog. The app can call any OpenAI-compatible chat-completions endpoint. For a hosted provider:

```sh
AI_PROVIDER=openai-compatible AI_API_KEY="your-key" AI_MODEL="your-model" npm run dev
```

For a self-hosted Ollama model on the same machine as Node, first install/run Ollama and pull a model, then use:

```sh
AI_PROVIDER=ollama AI_MODEL="your-local-model-tag" npm run dev
```

Ollama defaults to `http://127.0.0.1:11434/v1`; override it with `AI_BASE_URL` if needed. This supports **self-hosting open weights**, not training or fine-tuning a new model. A model service must be reachable from the machine running the Node backend; an Arena preview's server-side `127.0.0.1` is not the user's laptop. Optional settings are `AI_BASE_URL` (hosted providers default to `https://api.openai.com/v1`), `PORT` (defaults to `4173`), and `HOST` (defaults to `0.0.0.0`). Hosted API keys stay in the server environment and are never embedded in frontend code. The AI option appears when a compatible provider is configured and still requires user opt-in. An opted-in request sends the project summary, prompt, and enabled public KiCad metadata to that provider. Without opt-in, project details are not sent to the model. The integration panel's **Test AI** action sends a fixed prompt without project data. Public KiCad lookup uses only the text brief and can be unchecked separately. The model currently reviews/recommends; only the included local templates modify the schematic. Arbitrary model-generated netlists and automatic wiring are not implemented, because those require strict schema, pin-map, electrical-rule, and validation layers.

## Important engineering scope

This is a polished interactive prototype, **not a production-validated ECAD tool**. It does not claim full electrical correctness, support for every listed board revision, analog SPICE, instruction-set MCU simulation, true PCB autorouting, a comprehensive ERC/DRC engine, cloud sync, multi-user collaboration, firmware compilation/flashing, or a vendor-verified comprehensive component/footprint database. Most starter records are catalog entries and their package/pin metadata is indicative. Board/Gerber outputs use approximated placement/routing and unverified connectivity. Validate schematics and layouts with exact manufacturer datasheets and a proper toolchain (for example, KiCad and a suitable simulator) before connecting hardware or ordering boards.

## Source layout

```text
index.html              App shell
server/index.js         Dependency-free Node HTTP server and API
src/main.js             UI, input handling, persistence, modal flows
src/model.js            Project schema, starter design, parts catalog, pin anchors
src/render.js           SVG schematic and PCB preview rendering
src/assistant.js        Local design templates, firmware scaffold, explanations
src/simulator.js        Structural checks and behavioral preview
src/exporters.js        JSON, CSV, SVG, KiCad skeleton and draft manufacturing files
src/styles.css          Themes, responsive layout, component/workspace styling
test/                   Node test suite
```
