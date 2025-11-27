---
title: "Fixing Math Rendering in Hugo with PaperMod Theme"
date: 2025-11-27T12:45:00-06:00
draft: false
tags: ["hugo", "katex", "math", "papermod", "troubleshooting"]
categories: ["tutorials"]
author: "Ayush"
showToc: true
description: "A guide to fixing KaTeX math rendering issues in Hugo blogs using the PaperMod theme"
math: true
---

## The Problem

I was trying to add some math equations to my Hugo blog, but they simply would not render. Instead of seeing nicely formatted equations, I was getting the raw LaTeX syntax displayed as plain text on the page.

For example, I wanted to display Einstein's famous equation inline like this: $E = mc^2$

And a block equation like this:

$$\sum_{i=1}^{n} x_i = x_1 + x_2 + ... + x_n$$

But what I was seeing was just the literal text with dollar signs showing up on the page. Not exactly the elegant mathematical typography I was hoping for.

## Initial Setup

I had already done what I thought was the correct setup. In my `layouts/partials/extend_head.html`, I had added the KaTeX scripts:

```html
{{ if or .Params.math .Site.Params.math }}
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.css" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/katex.min.js" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.16.9/dist/contrib/auto-render.min.js" crossorigin="anonymous"
    onload="renderMathInElement(document.body, {
        delimiters: [
            {left: '$$', right: '$$', display: true},
            {left: '$', right: '$', display: false}
        ],
        throwOnError: false
    });"></script>
{{ end }}
```

I also had `math: true` in both my site's `hugo.yaml` under `params` and in the front matter of posts where I wanted math support. The scripts were loading correctly in the browser, I could verify that in the developer tools. Still nothing was rendering.

## The Root Cause

After some digging, I found the issue. Hugo uses Goldmark as its Markdown processor, and Goldmark was treating my `$` and `$$` delimiters as regular text. It was escaping them before KaTeX ever had a chance to process them.

The problem was not with KaTeX at all. It was with how Hugo was processing the Markdown content.

## What Did Not Work

I tried a few things that did not solve the problem:

1. Removing spaces between the `$` signs and the equation content
2. Putting the equation on a single line
3. Using different integrity hashes for the KaTeX CDN scripts
4. Various ways of calling `renderMathInElement`

None of these addressed the actual issue, which was happening at the Markdown processing stage, before the HTML was even generated.

## The Solution

Hugo has a feature called the `passthrough` extension for Goldmark. This extension tells the Markdown processor to leave certain delimiters alone and pass them through to the final HTML without any processing.

Here is what I added to my `hugo.yaml`:

```yaml
markup:
  goldmark:
    renderer:
      unsafe: true
    extensions:
      passthrough:
        enable: true
        delimiters:
          block:
            - - $$
              - $$
            - - \[
              - \]
          inline:
            - - $
              - $
            - - \(
              - \)
```

The key parts here are:

1. `passthrough.enable: true` activates the extension
2. The `delimiters` section defines which patterns should be passed through
3. `block` delimiters are for display math (centered, on its own line)
4. `inline` delimiters are for math within a paragraph

Pay attention to the YAML syntax here. The delimiter pairs are defined as nested lists. Each pair is a list of two items: the opening delimiter and the closing delimiter. The indentation matters.

## Testing the Fix

After updating the configuration, I restarted the Hugo server with:

```bash
hugo server --disableFastRender
```

The `--disableFastRender` flag ensures a complete rebuild, which is helpful when changing configuration files.

Now the math renders correctly. Inline equations like $a^2 + b^2 = c^2$ appear within the text flow, and block equations are displayed prominently:

$$\int_{a}^{b} f(x) \, dx = F(b) - F(a)$$

## Summary

If you are having trouble with math rendering in Hugo with the PaperMod theme, check these things in order:

1. Make sure you have the KaTeX scripts in your `extend_head.html` partial
2. Ensure `math: true` is set in your site config or post front matter
3. Add the `passthrough` extension configuration to your `hugo.yaml`
4. Restart the server with `--disableFastRender`

The passthrough extension is the crucial piece that many tutorials miss. Without it, Goldmark will mangle your math delimiters before KaTeX can process them.

---

## References

1. [Hugo PaperMod GitHub Issue #236 - How to enable Math Typesetting](https://github.com/adityatelange/hugo-PaperMod/issues/236)
2. [Hugo Discourse - Maths not rendered](https://discourse.gohugo.io/t/maths-not-rendered/47494/3)
3. [Hugo Documentation - Mathematics in Markdown](https://gohugo.io/content-management/mathematics/)
4. [KaTeX Auto-render Extension](https://katex.org/docs/autorender.html)
