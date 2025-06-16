# TailwindCSS Best Practices

- [Automatic Class Sorting with Prettier](https://tailwindcss.com/blog/automatic-class-sorting-with-prettier)

## 1. Sizing

- Wrap internal content in a container that defines layout and spacing ✨.
- Allow components to receive flexible children with grid or flex constraints ✨.

```tsx
const Card = () => (
  <div className="p-6 bg-white rounded shadow-md flex flex-col gap-4">
    <h2 className="text-xl font-bold">Title</h2>
    <p>Description</p>
  </div>
);

const TwoColumn = ({ children }: { children: React.ReactNode }) => (
  <div className="grid grid-cols-1 md:grid-cols-2 gap-6">{children}</div>
);
```

- Use min-height or max-height to provide limits without rigid constraints. Avoid hardcoding height unless necessary.
- Use `flex-grow` and `grid-auto-flow` appropriately. This allows components to expand or contract based on available space without breaking layout.

| Class              | Use Case                                                                   |
|--------------------|----------------------------------------------------------------------------|
| min-w-0            | Necessary in flex layouts when a child needs to shrink instead of overflow |
| max-w-\*, max-h-\* | Limit growing content (images, cards, modals), prevents overflow           |
| min-w-\*, min-h-\* | Enforce minimum touch targets or visible elements (buttons, input fields). |

### Flex Item

Each flex item (a direct child of a flex container) receives a final size based on:

- `flex-grow`: how much it grows when extra space is available.
- `flex-shrink`: how much it shrinks when space is limited.
- `flex-basis`: initial size before growing/shrinking.

Flexbox acts like a negotiator, redistributing space based on those values.

## 2. Layout 

- Use Flexbox for one-dimensional layouts, CSS Grid for two-dimensional layouts
- Responsive units (`rem`, `%`, `vh`, `vw`, or `clamp()`) over pixels

