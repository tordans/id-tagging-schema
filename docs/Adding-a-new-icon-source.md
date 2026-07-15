# Adding a new icon source (maintainers)

This document describes what has to happen when the tagging schema adopts icons from a **new family** (a new string prefix). See [`ICONS.md`](../ICONS.md) for families that are already allowed. It complements the contributor-facing overview there.

Adding a new icon family is a **curated decision**: discuss it with maintainers first (for example in an issue). We aim for a small, consistent set of map-related and general-purpose sources—not many overlapping general icon libraries.

## Is this a breaking change?

**Not for the JSON schema.** In [`schemas/preset.json`](../schemas/preset.json), `icon` is a free-form string, so values such as `roentgen-ford` need no schema update.

**It can break the user experience in apps** that render preset icons but do not yet resolve the new prefix. Existing presets may start using the new icons as soon as the changes are merged; apps should fall back when an icon is unknown (generic glyph, text only, etc.), but users may see missing icons until each consumer adds support.

Mention new icon families prominently in the [release notes](https://github.com/openstreetmap/id-tagging-schema/releases) and contact editor maintainers (see below)—that is usually more effective than relying on release notes alone.

## Rollout convention

When a new family is added:

- **Do not** immediately mass-replace icons on existing presets. Give data consumers time to implement the new source.
- **Do** use the new family for new presets, or for presets that only had a poor generic icon before.

If the set uses a restrictive license, document required attribution for data consumers (in the issue/PR and in [`ICONS.md`](../ICONS.md)).

## End-to-end flow

1. **Agree on a prefix** (e.g. `roentgen-`) and ensure **stable public SVG URLs** (GitHub raw, npm package, flat per-icon files, etc.).
2. **Document and wire up the family**: add it to [`ICONS.md`](../ICONS.md); add a Taginfo `icon_url` mapping in [`scripts/lib/build.js`](../scripts/lib/build.js); optional test fixture in [`scripts/lib/__tests__/schema-builder.test.js`](../scripts/lib/__tests__/schema-builder.test.js).
3. **Release** a new tagging-schema version.
4. **Contact maintainers** of each editor or other tool that renders preset icons so they can extend icon resolution or bundling (see below). Link them to the release and this document.
5. **Each consumer** implements support in its own codebase (iD, StreetComplete, GoMap!!, …).
6. **Use icons in data**: set `"icon": "prefix-name"` (or field `icons`) in [`data/`](../data/) presets, categories, or fields—**after** consumers have had a chance to catch up, following the rollout convention above.

## 1. Documentation and build

| What | Details |
|------|---------|
| [`ICONS.md`](../ICONS.md) | Add a row under “Where do the icons come from?” with link, prefix, license/attribution notes, and short description. |
| [`scripts/lib/build.js`](../scripts/lib/build.js) | In `generateTaginfo`, add an `else if` for the new prefix so Taginfo preset items get a correct `icon_url` (same idea as `maki-`, `temaki-`, `fas-`, `roentgen-`, `pinhead-`, `iD-`). |
| Tests | Optional: extend `scripts/lib/__tests__/schema-builder.test.js` with a fixture icon to assert Taginfo output. |
| Data | [`data/presets/**/*.json`](../data/presets), categories, fields—set `icon` / `icons` to the new prefixed id (last step; see flow above). |
| Build | `npm run build` / `npm run dist`. No SVG packages are added to the repository; only strings in JSON. |

The build also writes [`dist/interim/icons.json`](../dist/interim/icons.json) listing icons in use—useful for consumers that prefetch or bundle glyphs.

## 2. iD and Rapid

iD shows preset icons from **SVG symbol sprites** built at release time. Sprite generation and preset icon resolution are maintained in the [iD](https://github.com/openstreetmap/iD) project.

| Area | Where to look |
|------|----------------|
| Sprite build | [`package.json` scripts](https://github.com/openstreetmap/iD/blob/develop/package.json) (`dist:svg:maki`, `dist:svg:temaki`, …) |
| Sprite registration | [`modules/svg/defs.js`](https://github.com/openstreetmap/iD/blob/develop/modules/svg/defs.js) |
| Special sizing/styling | [`modules/ui/preset_icon.js`](https://github.com/openstreetmap/iD/blob/develop/modules/ui/preset_icon.js) (only if needed, e.g. Röntgen) |

[Rapid](https://github.com/facebook/Rapid) follows the same sprite approach where it mirrors iD.

## 3. StreetComplete

Preset icons are downloaded by the Gradle task [`DownloadAndConvertPresetIconsTask`](https://github.com/streetcomplete/StreetComplete/blob/master/buildSrc/src/main/java/DownloadAndConvertPresetIconsTask.kt) (registered as `downloadAndConvertPresetIcons` in [`app/build.gradle.kts`](https://github.com/streetcomplete/StreetComplete/blob/master/app/build.gradle.kts)).

StreetComplete maintainers extend `getDownloadUrls` with another prefix branch and URL template, then re-run that task. Unknown prefixes are skipped. The task expects **one HTTP URL per icon**, usually a **flat folder** of `.svg` files (as Maki, Temaki, and Röntgen publish). Sets that ship only bundled sprites need preprocessing first.

## 4. Other tools

Other consumers (e.g. [GoMap!!](https://github.com/bryceco/GoMap), [tagging-schema-browser](https://github.com/osmberlin/tagging-schema-browser)) need their own icon resolution or bundling support. See [`MIGRATION_GUIDE.md`](../MIGRATION_GUIDE.md) for notes aimed at schema consumers. A [bundling proposal](https://github.com/openstreetmap/id-tagging-schema/issues/2208) for shipping icons with the release artifacts is under discussion but not implemented yet.

## Checklist

- [ ] Maintainer discussion / issue for the new family
- [ ] Prefix and stable per-icon SVG URLs confirmed
- [ ] [`ICONS.md`](../ICONS.md) updated
- [ ] [`scripts/lib/build.js`](../scripts/lib/build.js) Taginfo mapping added
- [ ] Release with release-note call-out
- [ ] Editor / tool maintainers contacted
- [ ] Consumer apps updated (iD, StreetComplete, …)
- [ ] Preset `icon` values added gradually in `data/` (no mass replacement)
