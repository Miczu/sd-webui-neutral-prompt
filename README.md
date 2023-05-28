# Neutral Prompt

Neutral prompt is an a1111 webui extension that enhances the webui's CFG denoising algorithm. It replaces the original algorithm with a higher-quality implementation based on more recent research.

## Features

- [Perp-Neg](https://perp-neg.github.io/) CFG algorithm, invoked using the `AND_PERP` keyword
- saliency-aware noise blending, invoked using the `AND_SALT` keyword (credits to [Magic Fusion](https://magicfusion.github.io/) for the algorithm used to determine SNB maps from epsilons)
- standard deviation based CFG rescaling (Reference: https://arxiv.org/abs/2305.08891)

## Usage

### Keyword `AND_PERP`

The `AND_PERP` keyword, standing for "PERPendicular `AND`", integrates the Perp-Neg CFG algorithm. Essentially, `AND_PERP` allows for prompting concepts that highly overlap with regular prompts, by negating contradicting noise.

You could visualize it as such: if `AND` prompts are "greedy" (taking as much space as possible in the output), `AND_PERP` prompts are opposite, relinquishing control as soon as there is a disagreement in the generated output.

### Keyword `AND_SALT`

Saliency-aware noise blending is made possible using the `AND_SALT` keyword, shorthand for "SALienT `AND`". In essence, `AND_SALT` monitors high noise activity during denoising and dominates any high-activation regions in the output.

Think of it as a territorial dispute: the noise generated by the `AND` prompts is one country, and the noise(s) generated by `AND_SALT` prompts represent neighbouring nations. They're all vying for the same land - whoever strikes the strongest at a given time and location claims it.

## Examples

### Using the `AND_PERP` Keyword

Here is an example to illustrate one use case of the `AND_PREP` keyword. Prompt:

`beautiful castle landscape AND monster house castle :-1`

This is an XY grid with prompt S/R `AND, AND_PERP`:

![image](https://github.com/ljleb/sd-webui-neutral-prompt/assets/32277961/29f3cf34-2ed4-45d2-b73a-b6fadec21d61)

Key observations:

- The `AND_PERP` images exhibit a higher dynamic range compared to the `AND` images.
- The `AND` images sometimes struggle to depict a castle, a challenge not encountered by the `AND_PERP` images.
- The `AND` images tend to lean towards a purple color, because this was the path of least resistance between the two opposing prompts during generation. In contrast, the `AND_PERP` images, free from this tug-of-war, present a clearer representation.

### Using the `AND_SALT` Keyword

The `AND_SALT` keyword introduces a unique process called saliency-aware noise blending. It spotlights and accentuates areas of high-activation in the output.

Consider this example prompt utilizing `AND_SALT`:

```
a vibrant rainforest with lush green foliage
AND_SALT the glimmering rays of a golden sunset piercing through the trees
```

In this case, the extension identifies and isolates the most salient regions in the noise of the sunset prompt. Then, the extension applies this salient noise to the noise of the rainforest prompt. Only the portions of the rainforest noise that coincide with the salient areas of the sunset noise are affected. These areas are replaced by noise from the sunset prompt.

This is an XY grid with prompt S/R `AND_SALT, AND, AND_PERP`:

![xyz_grid-0008-1564977627-a vibrant rainforest with lush green foliage_AND_SALT the glimmering rays of a golden sunset piercing through the trees](https://github.com/ljleb/sd-webui-neutral-prompt/assets/32277961/2404f20b-47f6-457f-b4c5-76b9fd919345)

Key observations:

- `AND_SALT` behaves more diplomatically, enhancing areas where its impact makes the most sense and aligning with high activity regions in the output
- `AND_PERP` will find its way through anything not blocked by the regular prompt
- `AND` gives equal weight to both prompts, creating a blended result

## Advanced Features

### Nesting AND_PERP prompts

The extension enables nesting prompts orthogonal to other perpendicular prompts:

```
a red hot desert canion
AND_PERP [
    cold blue everest montain
    AND a beautiful woman climbing a massive rocks wall :1.1
    AND_PERP far away, ugly, ants, water :-0.6
] :0.9
AND a rocky sahara climbing party :0.7
```

To generate the final noise from the diffusion model, the extension will:

1. Use the noise generated from the prompt `far away, ugly, ants, water :-0.6`
2. Make it orthogonal to the combination of `cold blue Everest mountain :1` and `a beautiful woman climbing a massive rock wall :1.1`
3. Add this orthogonal noise to the combined prompts it was orthogonalized against
4. Make the resulting noise orthogonal to the combination of `a red hot desert canyon :1` and `a rocky Sahara climbing party :0.7`
5. Add this orthogonal noise (multiplied by 0.9) to the combined prompts it was orthogonalized against

The final single noise map is composed of all normal `AND` prompts combined with all orthogonalized `AND_PERP` prompts.

Each instance of the `AND_PERP` keyword delineates a separate denoising space within its square brackets `[...]`. Prompts inside it merge into a single noise map before further processing down the prompt tree.

Experimental evidence suggests that it's unnecessary to go beyond a depth of 2. We're still exploring if this adds to the precision of the generations. If you discover innovative ways of controlling the generations using nested `AND_PERP` prompts, please share in the discussions!

![image](https://github.com/ljleb/sd-webui-neutral-prompt/assets/32277961/f6d0c95b-8efd-4ce2-b5e4-928597facd34)

## Known issues

- The webui doesn't support composable diffusion via [`AND`](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Features#composable-diffusion) for samplers DDIM, PLMS, and UniPC. As Perp-Neg relies on composable diffusion, the extension will revert to the unmodified sampler implementation when these are used.

## Special Mentions

Special thanks to these people for helping make this extension possible:

- [Ai-Casanova](https://github.com/AI-Casanova) : for sharing mathematical knowledge, time, and conducting proof-testing to enhance the robustness of this extension
