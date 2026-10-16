# AI-Based Crop Recommendation for Farmers — implementation and design plan

## Product and implementation

A responsive farmer-first decision-support website for Indian crop planning. The main page will let a farmer build a field profile by choosing one of eight major Indian soil groups, then adjust pH, N, P, K, annual rainfall, temperature, season, monsoon outlook, and irrigation. Soil presets fill typical pH and nutrient starting values but remain editable. Rainfall outlook and irrigation change the estimated water available to the field.

A deterministic, explainable scoring engine will rank 16 crops on soil type, pH, N, P, K, rainfall, temperature, and season; display the top five with match percentages; and break down factors that help or lower each crop's match. Rule-based agronomy notes give explicit next steps such as lime for acidic soil, gypsum for alkaline soil, or nutrient supplementation. The server-side Manus AI service will add farmer-friendly explanations when available; the product must retain transparent scores and useful rule-based explanations if AI is unavailable.

A farmer-reference feed will show other submitted recommendations filtered by selected soil type. Farmers can choose to share a result through an explicit share action. Shared entries include the complete field profile, estimated water, and the top five crop matches; consent previews all of these fields, while district, display label, and a note are optional. The server recomputes the selected crop's score before storing and explaining results. Shared entries are stored in the project's managed MySQL database and returned through public tRPC procedures. When a successful query has no submissions, the feed may include clearly marked illustrative example entries; query failures must not be presented as live reports. Do not write demo seed rows to the shared managed database.

## Technical architecture

- Frontend: React and TypeScript in `client/src`, styled with the existing Tailwind setup. Keep the entry workflow on `/` and use small, focused source modules for soil/crop data, recommendation math, and UI.
- Recommendation data and matching: a pure TypeScript module shared by UI and server, with explicit profile assumptions and normalized per-factor scoring. In-range values are graded by proximity to the center of their broad planning ranges so match percentages remain comparative estimates, not success probabilities. Provide focused automated tests for the scorer and rainfall-water estimator.
- Backend: existing Express/tRPC server. Public procedures expose filtered farmer experiences and accept sanitized share submissions; input validation and bounds checking prevent malformed profile data. AI explanations are generated server-side through the configured Manus LLM service from server-recomputed recommendations and gracefully fall back to the local agronomy explanation rules.
- Persistence: add an additive Drizzle table and a committed migration for farmer-shared crop recommendation responses. The managed development and production database are shared; never use destructive migration or test writes, and do not seed pretend community activity.
- Runtime: use the existing configured `pnpm dev` runtime on port 3000. Publish route manifest for the current `/` page. Keep API secrets server-side; no user login is required for the initial experience.

### Main modules

- `client/src/pages/Home.tsx`: landing/workspace composition, profile inputs, recommendation results, and community references.
- `client/src/components/`: reusable farmer-friendly form, crop result, and feed components as needed.
- `client/src/lib/`: profile data and pure recommendation/scoring utilities.
- `server/routers.ts` and a focused `server/` feature module: validated public share/feed APIs and optional AI reasoning endpoint.
- `drizzle/schema.ts` and `drizzle/`: additive persistence schema and migration for shared results.
- `client/public/manus-routes.json`: declared page routes, served by the Vite frontend root.

## Design — Monsoon Ledger (approved)

- **Design movement:** Contemporary agrarian editorial, drawing on Indian rainfall records, agronomy survey ledgers, and field notebooks.
- **Core principles:** (1) Scientifically legible without intimidating terminology. (2) Every recommendation is explainable, not a black box. (3) Inputs are large and forgiving for touch use. (4) Community examples are labeled and grounded.
- **Color philosophy:** Deep rain blue-green (#164E63) signals dependable water and analytical trust; vivid crop green (#65A30D) marks healthy matches and forward action. Pale paper backgrounds and quiet field-line neutrals protect readability rather than compete with information.
- **Layout paradigm:** Editorial dashboard with an asymmetric, wide field-profile workbench and a persistent visual hierarchy: field context first, ranked crop ledger next, local farmer notes alongside/below. Avoid a centered uniform tile grid; use varied, content-led rows and a results rail.
- **Signature elements:** (1) Rainfall ledger bars and fine ruled dividers. (2) Compact circular/strip match indicators. (3) Small field-note labels and nutrient chips that make data feel practical.
- **Interaction philosophy:** Selecting a soil profile immediately populates its typical values; editable controls update recommendations in place. Changes remain visible, accessible, and reversible; sharing is deliberate and previews what will be shared.
- **Animation:** Restrained 150–220 ms transitions for preset selection, match bars, accordion details, and feedback. Use reduced-motion preferences. Never delay core form or result access behind animation.
- **Typography system:** Hind for a locally familiar, clear interface voice; a sturdy Hind semibold for headings and readable Hind regular for body and controls. Use tabular numerals for NPK, rainfall, and percentages; clear type scale and generous line height.
- **Brand essence:** A practical field-to-harvest guide that helps Indian farmers compare crop choices using their soil and season; **grounded, clear, encouraging**.
- **Brand voice:** Direct, supportive, and specific; pair numbers with a short reason. Examples: “Your field profile is ready. Let’s find crops that fit this season.” “Strong rainfall fit; add nitrogen before sowing for a better start.”
- **Wordmark & logo:** “AI-Based Crop Recommendation for Farmers” paired with a simple seed-in-furrow mark: two field rows forming a sprout above a rainfall-ledger baseline, drawn in the signature deep rain blue-green.
- **Signature brand color:** Deep monsoon teal, #164E63.

## Hosting and dependencies

Reuse installed React, TypeScript, tRPC, Drizzle/MySQL, and existing Lucide icon dependencies. Use Manus-provided server-side AI credentials if present in the runtime; no new third-party key is assumed. Use existing managed database and server capabilities. This product is advisory decision support, not a substitute for local agricultural extension services or soil-lab analysis; label typical presets and estimated water as estimates.
