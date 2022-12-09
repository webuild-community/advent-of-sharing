---
title: You can’t change the stacking context, but you can change the stacking context
author: huytd
date: 12-09-2022
---
In our product's new design, we have a content card with a backdrop blur effect, pretty much like this:

```typescript
<div class="card">
    some content
</div>

.card {
    backdrop-filter: blur(20px);
    background: rgba(255, 255, 255, 0.7);
}
```

One problem that comes up is: The `backdrop-filter` property will create a new stacking context. This means if we have multiple cards placed next to each other, each card will be rendered on top of the other's content.

For example, if one card has a dropdown component and another card is placed next to it:

```typescript
<div class="card">
    <dropdown-component></dropdown-component>
</div>

<div class="card">
    the next card
</div>
```

When the dropdown is expanded, it's rendered under the next card. This is how stacking context works.

To mitigate this issue, the solution is: Not applying the `backdrop-filter` effect on the `.card` element but creating a new layer for the backdrop inside the card, so when the new stacking context is created, it will not affects any of the adjacent elements.

```typescript
<div class="card"></div>

.card {
    &::before {
        content: " ";
        display: block;
        position: absolute;
        inset: 0;
        backdrop-filter: blur(20px);
        background: rgba(255, 255, 255, 0.7);
    }
}
```
