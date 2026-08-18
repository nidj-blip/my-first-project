# Luna & Fluffy — Character Sheet Prompt Package

Premium children's storybook character reference for **Luna** (a joyful 3-year-old girl) and
**Fluffy** (an enormous silver Maine Coon cat), in a Disney/Pixar-inspired stylized-3D look
blended with watercolor & gouache storybook textures.

Built with a slot-based prompt architecture (composition → identity → face → eyes → hair →
render module → body → wardrobe → lighting → quality tail → negative tail) so every detail is
specific and carries forward unchanged across views — consistency is the product.

---

## Character bible (canonical details — carry forward on every iteration)

### Luna
| Trait | Canonical description |
|---|---|
| Age / build | Joyful 3-year-old girl, natural toddler proportions, round soft cheeks |
| Eyes | Large expressive round amethyst-purple eyes, long soft lashes, warm natural catchlights |
| Face | Button nose, freckles dusted across the nose and upper cheeks, bright joyful smile with tiny baby teeth |
| Hair | Long chestnut-brown hair tied into two tight braided pigtails *(brief didn't specify a color — chestnut-brown is the working choice; swap freely)* |
| Hair accessories | Soft lavender satin ribbons at the end of each braid, small sparkling amethyst pins adorning the braids |
| Outfit | Two-piece haute-couture light-lavender outfit: a fitted cropped jacket with delicate tonal embroidery and covered buttons, and a matching flared skirt with soft petal-like hem |
| Feet | Crisp white low-top sneakers, ankle socks with printed little owls |
| Personality | Friendly, joyful, curious |

### Fluffy
| Trait | Canonical description |
|---|---|
| Breed / scale | Enormous fluffy silver Maine Coon cat, standing shoulder-height to Luna |
| Eyes | Amethyst-purple eyes matching Luna's, gentle expression |
| Fur | Luxurious long layered silver fur with soft white ruff at the chest |
| Tail | Giant feathered plume tail |
| Ears | Tall tufted lynx-tip ears |
| Presence | Majestic yet adorable, calm and protective beside Luna |

---

## Master prompt (full character sheet — turnaround + expressions)

> Character reference sheet composition, top row three consistent full-body views evenly spaced — front view, side view, and back view — of the same two original characters standing together, bottom row a set of head-and-shoulders expression portraits of the girl (happy, surprised, laughing), identical original characters on all views, pure white seamless studio background, professional character sheet presentation, a joyful 3-year-old girl with natural toddler proportions and round soft cheeks, button nose, freckles dusted across her nose and upper cheeks, bright joyful smile, large expressive round amethyst-purple eyes with long soft lashes and warm natural catchlights, long chestnut-brown hair tied into two tight braided pigtails finished with soft lavender satin ribbons and small sparkling amethyst pins adorning the braids, wearing a two-piece haute-couture light-lavender outfit with a fitted cropped embroidered jacket and matching flared skirt with a soft petal hem, crisp white low-top sneakers, ankle socks printed with little owls, no bag, standing beside an enormous fluffy silver Maine Coon cat that reaches her shoulder, the cat with amethyst-purple eyes, a gentle expression, luxurious long layered silver fur with a soft white chest ruff, a giant feathered plume tail and tall tufted lynx-tip ears, majestic yet adorable, high-end stylized 3D animation render with soft subsurface-scattering skin glow, expressive large stylized eyes, finely detailed soft hair strands, plush fur and cloth simulation, blended with delicate watercolor and gouache paper textures, soft pastel palette, premium children's storybook illustration style inspired by modern animated features without copying any existing character, warm cinematic three-point lighting with soft global illumination and a gentle rim light, shallow depth of field, ultra detailed, high-end 3D animation studio quality, consistent character design across all views, clean white background, 4K, exactly two characters — the girl and the cat — and no other people, no duplicate stray figures outside the labeled views, no text, no watermark, no logos, no frame borders, no background props or furniture, no distorted anatomy, no extra fingers, original characters only, not resembling any real person or existing copyrighted character

**Midjourney parameter line (as briefed):**
`--chaos 5 --ar 9:16 --raw --profile kidcig9 --stylize 900 --v 7`

Note: 9:16 vertical suits a stacked two-row sheet (turnaround on top, expressions below). If
the model crowds the panels, run 16:9 or 3:2 instead, or split into the panel prompts below.

---

## Panel prompts (higher reliability — one panel per generation)

Multi-view sheets are the most fragile composition; these split the sheet into consistent
single-panel renders. Keep the identity block identical in every prompt.

**Shared identity block** (paste into every panel prompt where `[IDENTITY]` appears):

> a joyful 3-year-old girl with natural toddler proportions and round soft cheeks, button nose, freckles across her nose, large expressive round amethyst-purple eyes with long soft lashes, long chestnut-brown hair in two tight braided pigtails with soft lavender satin ribbons and small sparkling amethyst pins, wearing a two-piece haute-couture light-lavender outfit with fitted cropped embroidered jacket and flared petal-hem skirt, white low-top sneakers and owl-print ankle socks, beside an enormous fluffy silver Maine Coon cat at her shoulder height with amethyst-purple eyes, gentle expression, luxurious long silver fur with white chest ruff, giant feathered plume tail and tufted lynx-tip ears

**Shared style + negative tail** (append to every panel prompt):

> high-end stylized 3D animation render, soft subsurface-scattering skin glow, expressive large stylized eyes, detailed soft fur and cloth simulation, watercolor and gouache paper textures, soft pastel palette, premium children's storybook illustration, warm cinematic lighting, soft global illumination, gentle rim light, shallow depth of field, ultra detailed, 4K, clean pure white seamless background, consistent character design, exactly two characters and no other people, no text, no watermark, no logos, no frame borders, no props, no distorted anatomy, original characters only, not resembling any existing copyrighted character

1. **Front view:** `Full-body front view of two original characters standing side by side facing the camera, both feet flat on the ground, head-to-toe framing with the whole body visible and not cropped, [IDENTITY], [STYLE + NEGATIVE TAIL]`
2. **Side view:** `Full-body side-profile view of the same two original characters standing side by side facing left, head-to-toe framing not cropped, [IDENTITY], [STYLE + NEGATIVE TAIL]`
3. **Back view:** `Full-body back view of the same two original characters standing side by side facing away from the camera, the girl's braided pigtails and ribbons clearly visible from behind, the cat's giant plume tail raised, head-to-toe framing not cropped, [IDENTITY], [STYLE + NEGATIVE TAIL]`
4. **Expressions:** `Expression sheet, a row of three head-and-shoulders portraits of the same original little girl — happy warm smile, wide-eyed surprised with a small open mouth, and mid-laugh with eyes crinkled — evenly spaced on a pure white background, [IDENTITY], [STYLE + NEGATIVE TAIL]`

---

## Iteration rules

- Every established detail above carries forward **unchanged** unless explicitly changed —
  restate the full identity block each run so nothing silently drifts.
- Original characters only: "Disney-Pixar inspired" is a mood, never a likeness of any
  existing character.
- Targeted fix ideas for round two: tighten the wardrobe embroidery detail, enlarge the cat
  relative to Luna, push more gouache grain into the backgroundless white, or swap the
  expression row for a pose row (waving, hugging the cat, twirling).
