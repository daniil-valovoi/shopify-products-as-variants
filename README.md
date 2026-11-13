# Shopify Products as Variants

Make **separate Shopify products behave like variants** on the product page, without Shopify Plus or an app.

"Cotton Shirt – Blue" and "Cotton Shirt – Yellow" stay independent products (own URL, SEO, images, description), but the storefront shows one picker: Color: Blue / Yellow, plus Size etc.

Shopify's native Combined Listings app is [Plus-only](https://shopify.dev/docs/apps/build/product-merchandising/combined-listings); this is a free alternative for any plan.

## How it works

1. A product-list metafield `custom.product_variations` (type `list.product_reference`) stores the sibling products.
2. A Shopify Flow, triggered on product creation, finds all products sharing a `sku-…` tag and writes every sibling into each product's metafield.
3. The theme's variant picker loops over the metafield products and appends their linking-option value as extra choices. Each extra choice carries `data-connected-product-url`, so the theme's existing variant-picker JS navigates to that product (the same mechanism Shopify uses for combined listings).

No new JS or APIs: it reuses the theme's existing picker behavior.
Note: the provided snippet is Horizon theme-specific. It also may contain code incompatible with your theme. Info on how to add this functionality to your theme is below.

## Setup

### 1. Create the metafield
Settings → Custom data → Products → Add definition
- Namespace and key: `custom.product_variations`
- Type: **Product**, **List of products**
- Enable Storefront access (needed for Liquid).

### 2. Create the products
One product per value of the linking option (e.g. one per color):
- `Cotton Shirt - Blue`, `Cotton Shirt - Yellow`, …
- **Option 1 = linking option** (e.g. `Color`), with exactly **one value** per product (that product's color). Drag it to the **top** in the product editor.
- Options 2 and 3 (e.g. `Size`, `Material`) can have as many values as needed.
- Shopify allows max 3 options per product, so up to 3 option types total, one of them being the linking option.
- The linking option **name must be identical** across all products in the group (`Color`, not `Colour`).

### 3. Tag the group
Give every product in the group the same tag, starting with `sku-`, e.g. `sku-cotton-shirt`.

### 4. Import the Flow
Shopify Flow app → Import → select `products-as-variants.flow`.
- Trigger: product added
- Reads the first `sku-*` tag → fetches products with that tag (max 100) → writes the other products' IDs into each one's `custom.product_variations`.

Add tags **before or at** product creation (the trigger fires on creation). To re-link existing products, re-run the flow manually (select the product in Products tab > click on 3 dots button > Run Flow automation).

### 5. Patch the theme
Add the code from `modified-variant-picker.liquid` to your variant picker snippet. Written for Shopify's Horizon theme (`snippets/variant-picker.liquid`); two insertion points (buttons/swatches and dropdown). Only the first option gets the extra values. 2nd and 3rd options render normally per each variant.

## Using another theme
The provided snippet is made for Horizon theme, but the idea is simple and universal. In your theme's variant picker snippet, for the **first option** only, after the standard values loop:

```liquid
{% for p in product.metafields.custom.product_variations.value %}
  {% assign opt = p.options_with_values | where: 'name', option.name | first %}
  {% for v in opt.values %}
    <!-- render a choice for v.name that links to {{ p.url }} -->
  {% endfor %}
{% endfor %}
```

Each extra choice must navigate to `p.url` (or whatever your theme's JS does for connected products).

## Limitations
- Flow fetches up to 100 products per group.
- Products are linked on creation; edits to tags later don't re-link automatically, unless configured in the Flow.
- The Flow sometimes doesn't see newly created products. If this issue is observed, run the Flow manually for at least one product from the group.
- Linking option must be option 1 and single-valued per product.
- Option values are matched by name across products.

## Support
If this repo was useful, you can help me by starting your next Shopify store through
[my affiliate link](https://shopify.pxf.io/3-months). You get the latest free trial offer Shopify has at the given moment, and I receive a small commission at no extra cost to you. Thanks!

## Disclaimer
This project is not affiliated with, endorsed by, or sponsored by Shopify Inc. Shopify and Horizon are trademarks of Shopify Inc. Use at your own risk.