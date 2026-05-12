# Amplicomm – Product Listing & Collection Architecture

# 1. How Featured vs Non-Featured Products Were Loaded & Separated

The collection products were loaded using Shopify’s native `collection.products` object inside the `main-collection-product-grid.liquid` section.

```liquid
{%- paginate collection.products by section.settings.products_per_page -%}
```

### Separation Logic

Featured and non-featured products were handled logically through collection conditions and rendering flow.

### Approach Used

* Featured products can be identified using:

  * Product tags
  * Product metafields
  * Manual collection rules
  * Specific sorting priorities

### Rendering Strategy

The product grid loops through products using:

```liquid
{%- for product in collection.products -%}
```

This structure allows:

* Featured products to render first
* Remaining products to render afterward
* Flexible future customization without changing the core architecture

### Benefits

* Clean separation of logic
* Easy scalability
* Better merchandising control
* Compatible with Shopify filtering & sorting

---

# 2. How Infinite Scroll Was Implemented

Infinite scrolling was implemented using Shopify pagination combined with a custom “Load More” mechanism.

### Pagination Setup

```liquid
{%- paginate collection.products by section.settings.products_per_page -%}
```

### Conditional Load More Button

```liquid
{%- if paginate.pages > 1 -%}
  {% if section.settings.scroll_enabled %}
    {%- if paginate.next -%}
      Load More
    {%- endif -%}
  {% endif %}
{%- endif -%}
```

### Implementation Flow

1. Initial products are rendered server-side.
2. Next page URL is obtained from `paginate.next`.
3. JavaScript fetches the next page asynchronously.
4. New product cards are appended to the existing grid.
5. Process repeats until no more pages remain.

### Why This Approach?

* Better UX compared to full page reloads
* Reduces initial payload
* Improves perceived performance
* Works well with Shopify pagination limitations

---

# 3. How Duplicate Products Were Prevented

Duplicate prevention was handled during the append/render process.

### Strategy Used

Each product card uses Shopify’s unique product identifier.

### Prevention Logic

Before appending a product:

* Existing rendered product IDs are checked
* Duplicate IDs are skipped
* Only unique products are injected into the DOM

### Why It Matters

Infinite scroll implementations can accidentally reload overlapping products when:

* Filters change
* Sorting changes
* Pagination resets
* Multiple asynchronous requests occur

Using unique product identifiers ensures:

* Stable rendering
* No repeated product cards
* Consistent customer experience

---

# 4. How the Solution Scales for Large Collections

The implementation was designed with scalability in mind.

### Key Scaling Decisions

## A. Shopify Pagination

```liquid
paginate collection.products by section.settings.products_per_page
```

Only a limited number of products are rendered initially.

This prevents:

* Large DOM sizes
* Heavy initial payloads
* Slow page rendering

---

## B. Lazy Loading

```liquid
{%- if forloop.index > 2 -%}
  {%- assign lazy_load = true -%}
{%- endif -%}
```

Images after the first few products are lazy loaded.

### Benefits

* Faster Largest Contentful Paint (LCP)
* Reduced bandwidth usage
* Improved mobile performance

---

## C. Modular Product Rendering

```liquid
{% render 'card-product' %}
```

Using reusable snippets improves:

* Maintainability
* Reusability
* Component-based architecture
* Easier debugging

---

## D. Incremental Product Loading

Products load progressively instead of rendering the entire collection at once.

This makes the implementation suitable for:

* Large Shopify catalogs
* Thousands of products
* Mobile-first storefronts

---

# 5. How Filtering & Sorting Were Handled

Filtering and sorting were implemented using Shopify’s native storefront filtering system.

### Sorting Logic

```liquid
{%- assign sort_by = collection.sort_by | default: collection.default_sort_by -%}
```

Available sort options:

```liquid
{%- for option in collection.sort_options -%}
```

### Filter Rendering

```liquid
{% render 'facets',
  results: collection,
  enable_filtering: section.settings.enable_filtering,
  enable_sorting: section.settings.enable_sorting,
  filter_type: section.settings.filter_type,
  paginate: paginate
%}
```

### Logic Maintenance Strategy

The implementation ensures:

* Filters persist during pagination
* Sorting remains consistent during infinite scroll
* URL parameters remain synchronized
* Product count updates correctly
* Empty states are handled properly

### Benefits

* Native Shopify compatibility
* SEO-friendly structure
* Cleaner URL handling
* Better storefront performance

---

# 6. Limitations of Liquid & How They Were Solved

Shopify Liquid has several limitations when building advanced storefront experiences.

## Limitation 1: No Native Infinite Scroll

### Problem

Liquid renders server-side and cannot dynamically fetch products on its own.

### Solution

Used JavaScript with Shopify pagination endpoints:

* Fetch next paginated page
* Parse returned HTML
* Append products dynamically

---

## Limitation 2: Limited Data Manipulation

### Problem

Liquid is not ideal for complex array manipulation or deduplication.

### Solution

Moved dynamic logic to JavaScript:

* Product deduplication
* Infinite scroll handling
* State management
* Dynamic DOM updates

---

## Limitation 3: Performance Issues with Large Collections

### Problem

Rendering too many products server-side increases page load time.

### Solution

Implemented:

* Pagination
* Lazy loading
* Incremental rendering
* Reusable snippets

---

## Limitation 4: Limited Reactive State Handling

### Problem

Liquid cannot maintain frontend reactive states.

### Solution

Used frontend JavaScript to:

* Track loaded pages
* Manage loading states
* Prevent duplicate requests
* Maintain infinite scroll behavior

---

# Technical Highlights

* Shopify Liquid based architecture
* Native Shopify filtering support
* Pagination-driven product loading
* Infinite scroll / Load More support
* Lazy loaded images
* Modular reusable snippets
* Scalable large collection handling
* Optimized storefront performance

