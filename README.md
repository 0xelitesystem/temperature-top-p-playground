# Temperature and top_p Playground

Drag temperature, top_k and top_p and watch a real softmax distribution reshape live, with every stage of the truncation pipeline shown and the edge cases explained.

**Live demo:** https://0xelitesystem.github.io/temperature-top-p-playground/

## Use

1. Pick a preset vocabulary (Confident, Flat / uncertain, Two-way tie, or Long tail), or edit the logits directly.
2. Drag Temperature, top_k, and top_p, or click an edge case button such as T = 0 or top_k = 1.
3. Read each pipeline stage and the Numbers panel.
4. In the sampling demo, set Samples and Seed and click Draw samples to compare observed counts with the probabilities.

## Why this exists

Sampling parameters are usually explained with one sentence about creativity. This tool shows the actual math stage by stage on a small vocabulary you can edit, in one HTML file with no tracking and no network calls, released under the MIT license.

## Features

- **Real math, computed in the page.** Softmax with temperature (subtract the max logit for numerical stability, divide by T, exponentiate, normalize), then top_k rank truncation, then top_p nucleus truncation, then renormalization. No libraries, no external dependencies, no network requests.
- **The pipeline is shown as ordered stages**, not implied. Raw logits, after temperature, after top_k, after top_p, and the final renormalized distribution each get their own bar chart. Tokens removed by a stage stay visible and are marked in text with which control cut them, so you can see the effect of each knob rather than inferring it from a shorter chart.
- **Editable vocabulary.** Change any logit, add tokens, remove tokens. Four presets teach different lessons: a confident distribution where one token dominates, a flat uncertain one, an exact two-way tie, and a long tail of near-identical low-probability tokens (the case where top_p and top_k behave very differently).
- **Numbers next to the picture.** Per-token probability before and after, the nucleus cumulative sum each token was compared against, the entropy of the final distribution in bits, the effective vocabulary size (perplexity, 2 raised to the entropy), how many tokens survived each stage, and how much probability mass each stage removed.
- **Sampling demo.** Draw N samples from the final distribution with a seeded generator and compare the observed frequency counts against the theoretical probabilities. The sample count is adjustable, so you can watch sampling variance shrink as N grows. Same seed and same N always give the same counts.
- **Edge cases handled and explained**, one button each: temperature 0 (treated as greedy argmax, never a division by zero), very small temperature, large temperature flattening toward uniform, top_p of 1.0, top_p smaller than the largest single probability (the nucleus still keeps at least one token), top_k of 1, top_k larger than the vocabulary, and exact ties.
- **An honest explainer** in plain language: why temperature 0 is still not fully deterministic in practice, why top_p and top_k interact, what is actually known about the order providers apply them in (and how much of it is undocumented), and why "raise temperature for creativity" is a crude description of what actually happens.
- Dark theme by default with a light toggle that persists, responsive down to 360px, keyboard accessible, copy buttons for JSON and CSV.

## How it works

The core numeric layer is a set of pure functions that take values and return a result object, with no DOM access. The pipeline runs in this order:

1. **Temperature.** The maximum logit is subtracted from every logit before dividing by T. That keeps `exp()` from overflowing at very small temperatures and changes nothing about the result, because softmax is invariant to a constant shift. A temperature of 0 or below branches to greedy argmax instead of dividing, which is the limit of the softmax as T approaches 0.
2. **top_k.** A rank cut. The tokens are ordered by descending probability with ties broken by ascending index, and everything past rank k is dropped. k is clamped to at least 1 and at most the vocabulary size.
3. **top_p.** A mass cut, measured against the distribution renormalized over the top_k survivors. That is what happens in the common implementation where top_k masks the losing logits to negative infinity and top_p then reads the softmax of what is left. Tokens are added in rank order until the running total reaches p; the comparison uses a small tolerance so floating point drift does not drop the last token, and a p of 1 or above short-circuits to keep-all the way reference implementations treat it. At least one token always survives.
4. **Renormalize.** The survivors are rescaled to sum to 1. This is the distribution a sampler actually draws from.

Entropy is the standard Shannon entropy in bits. The effective vocabulary size is the perplexity, 2 raised to that entropy, which is exactly the size of the uniform distribution with the same entropy. The sampling demo uses a seeded mulberry32 generator and inverse-CDF selection, so results are reproducible across browsers.

**The logits are invented and editable.** They are a toy vocabulary chosen so each lesson is visible on a handful of tokens instead of a hundred thousand. Nothing in this tool reports a measurement from any specific model, and no benchmark figures are quoted. Real vocabularies run from roughly 30,000 tokens to a little over 250,000 depending on the tokenizer (Llama 2 uses 32,000, Llama 3 uses 128,256, Gemma uses 256,000), so a real top_p nucleus on an uncertain step can hold hundreds of tokens where this page shows a few. The shape of the behavior is the same; the scale is not.

Note on ordering: providers differ, and most hosted APIs do not document their internal sampler order. This tool applies temperature, then top_k, then top_p, which is one common arrangement. The explainer in the page covers what changes when the order differs.

## Privacy

Everything runs in your browser. Nothing is uploaded, nothing is logged, and the page makes no network requests of any kind. There is no analytics, no tracking, and no external dependencies. The only thing stored is your light or dark theme preference, in your own browser's local storage.

## Run locally

```
git clone https://github.com/0xelitesystem/temperature-top-p-playground
cd temperature-top-p-playground
```

Open `index.html` in a browser, or serve the folder with `python -m http.server` and visit http://localhost:8000.

## Build

No build step. The whole tool is one `index.html` file.

## License

MIT. See [LICENSE](LICENSE).

## More

- Catalog of tools: https://0xelitesystem.github.io/
- https://elitesystem.ai
