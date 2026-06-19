---
applyTo: 'src/_areas/**,src/_cards/**,src/_levels/**,src/_customer_details/**'
---

# Model Structure

This Jekyll site implements a CodeMemo card deck model. The key collections are `_areas`, `_levels`, `_cards`, and `_customer_details`.

## Areas

Area files live in `src/_areas/<slug>.md`. The **filename stem** (without `.md`) is the area identifier — it must match the `area:` field in every card that belongs to it.

```yaml
---
title: "Display Name"
area:  "Display Name"   # human-readable, used in templates
weight: 50              # determines CSS colour class (see below)
color: 00e675           # hex without #, metadata only — not used by templates
---
```

### Area colour weights

The `weight` value becomes the CSS class `area-{weight}` on card elements. Pre-defined SCSS variables (in `src/_sass/model/_variables.scss`) map these to colours:

- `10` - `#fb001d` red
- `20` - `#172a08` dark green
- `30` - `#66459e` purple
- `40` - `#fa8b00` orange
- `50` - `#00e675` bright green
- `60` - `#00a8ff` blue
- `70` - `#d136f5` magenta
- `80` - `#424242` dark grey
- `90` - `#520303` dark red

The `color` hex in the area frontmatter is **ignored by templates** — only `weight` drives the colour.

## Levels

Level files live in `src/_levels/<slug>.md`. The level displayed for a card is derived from the card's numeric `level:` value using this formula:

```text
range  = 100 / number_of_levels   (= 20 with 5 levels)
index  = card.level / range       (integer division)
level  = levels_sorted_by_weight_descending[index]
```

With the standard 5 levels (weights 10→6):

- `0` - `19`: Novice
- `20` - `39`: Beginner
- `40` - `59`: Intermediate (Competent)
- `60` - `79`: Advanced
- `80` - `99`: Expert

Use round numbers such as `10`, `30`, `50`, `70`, `90` as canonical values.

## Cards

Card files live in `src/_cards/<area-slug>/<card-slug>.md`. Organising by area slug is a convention only — Jekyll picks up all files recursively.

```yaml
---
title:   Card Title          # max ~32 chars for clean display
caption: Short teaser line   # max ~60 chars
level:   50                  # numeric, see table above
area:    ecological          # must match the area filename stem
header:
  teaser: /assets/images/cards/codememo.png
---
```

Card content should contain:

1. Description (~600 chars) — elaborates the practice
2. `**Discussion:**` paragraph — broader perspective / rationale

## Customer details & score

`src/_customer_details/sample/score.md` holds gauge scores for each card. The key under `scores:` must exactly match the **card filename stem** (slug). When adding or replacing cards, update this file accordingly.

```yaml
scores:
  card-slug:   0 0 0 0   # Throughput Feedback Payback Complexity (0–3 stars each)
```

### Card annotations and snippets

`src/_customer_details/<client>/snippets/<area>/<card>.md` lets you extend an existing card without editing the card file itself.

- The snippet filename must match the card filename stem.
- The snippet content is rendered automatically below the card page and in report outputs that include snippets.
- Use this for observations, annotations, elaboration, recommendations, or any customer-specific note you want attached to the card.
- The folder name under `snippets/` is organizational only; matching happens on the filename stem.

`src/_customer_details/<client>/report-parts/*.md` assembles the report narrative. Those files can also include per-area findings by calling `report/get-area-findings.html` with an area slug.

## Replacing the model

When redefining areas and cards:

1. Create new area files in `src/_areas/`.
2. Create new card files in `src/_cards/<area-slug>/`.
3. Update `src/_customer_details/sample/score.md` with the new card slugs.
4. **Delete** the old area files and old card subdirectories — they will otherwise still appear in the built site.
5. Run `bundle exec rake jekyll:build` to verify.
