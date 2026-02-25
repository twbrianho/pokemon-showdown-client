# How to Add a Fakemon (Client Guide)

This guide explains how to add custom Teambuilder data and sprites to your deployed Pokémon Showdown UI.

For the game engine to actually allow the Fakemon in battle, you **must also follow the Server Guide** located in your server's `data/mods/dnd/` directory!

Because Pokémon Showdown heavily optimizes its UI by compiling all metadata into massive binary search arrays and caching chunks, we must inject our Fakemon into the local cache and recompile the client.

## 1. Inject Teambuilder Metadata (The Caches)

The Teambuilder reads from static Javascript files in the `play.pokemonshowdown.com/data/` folder, which are generated from the `caches/` directory.

### A. Add to Client Pokedex

Open `caches/pokemon-showdown/data/pokedex.ts` and add your Fakemon to the very top.

- **CRITICAL:** Use the exact same unique **negative** Pokedex ID (`num`) you used on the Server. Negative IDs instruct the Client UI to dynamically support your asset routing without hacking the core engine.

```typescript
export const Pokedex: import('../sim/dex-species').SpeciesDataTable = {
	mycustomfakemon: {
		num: -1001, // Must match the server!
		name: "MyCustomFakemon",
		types: ["Fire", "Fairy"],
		baseStats: { hp: 80, atk: 110, def: 80, spa: 110, spd: 80, spe: 100 },
		abilities: { 0: "Magic Guard", H: "Levitate" },
...
```

### B. Add to Client Learnsets

Open `caches/pokemon-showdown/data/learnsets.ts` and add the exact same move dictionary you defined on the server so the Teambuilder dropdown menus populate.

```typescript
export const Learnsets: import('../sim/dex-species').LearnsetDataTable = {
	mycustomfakemon: {
		learnset: {
			tackle: ["9L1"],
			flamethrower: ["9M"],
...
```

## 2. Add Sprites & Assets

Place your images into the local `play.pokemonshowdown.com/sprites/` folders using the all-lowercase, alphanumeric ID of your Fakemon (e.g., `mycustomfakemon`).

- **Gen 5 Fallback (PNG):** Used in the Teambuilder list, party icons, and older UIs.
    - Static Front: `sprites/gen5/mycustomfakemon.png`
    - Static Back: `sprites/gen5-back/mycustomfakemon.png`
- **Battle Animations (GIF):** Used in the actual battle screen.
    - Animated Front: `sprites/ani/mycustomfakemon.gif`
    - Animated Back: `sprites/ani-back/mycustomfakemon.gif`

## 3. Compile the Client

You must rebuild the client's data bundles so it can detect your new sprites (computing their dimensions automatically) and index your Fakemon into the Binary Search cache. Because the UI reads from the `dist/` JS files, you have to compile your `.ts` changes into JavaScript first.

1. Open a terminal in the `pokemon-showdown-client/caches/pokemon-showdown` directory.
2. Run the command: `npm run build`. This converts your Typescript additions into standard Javascript in the `dist` folder.
3. Switch back to the `pokemon-showdown-client` root directory.
4. Run the command: `node build full`
5. Notice in the console output that it says `Updating animated sprite dimensions... DONE` and `Building search-index.js... DONE`.

## 4. Test & Deploy

- **Local Testing:** You can test locally by running `npm run start` (which runs an `http-server` on port 8080).
- **Live Deployment:** For your friends to see the Fakemon, you must push the updated `play.pokemonshowdown.com/` directory up to your web host (e.g., Cloudflare Pages or GitHub Actions).
    - Specifically ensure `play.pokemonshowdown.com/data/` (the newly compiled search chunks) and `play.pokemonshowdown.com/sprites/` (your new images) are successfully deployed!
