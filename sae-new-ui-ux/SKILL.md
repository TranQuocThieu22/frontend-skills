---
name: sae-new-ui-ux
description: Outlines styling, UI component integration, Mantine v8 styling, custom CSS, animations, and design aesthetics in the SAE-new project. Use when designing, styling, animating, or tweaking components.
---

# SAE-New UI/UX and Styling Standards Skill

This skill explains how to build high-fidelity, polished, premium interfaces in the `sae-new` project using Mantine v8, custom CSS modules, global animations, and our dedicated design system.

## Typography and Spacing System (`FONT_FAMILY`, `FONT_SIZE`)

Always use the standard font families, sizes, weights, and letter-spacings imported from `@/shared/consts/font/font` instead of raw pixels or hardcoded fonts:

- **Primary Font**: `FONT_FAMILY.PRIMARY` (`Roboto`, `Nunito Sans`, etc.) - used for default UI labels and tables.
- **Display Font**: `FONT_FAMILY.DISPLAY` (`Be Vietnam Pro`, `Roboto`) - used for headers and titles.
- **Monospace Font**: `FONT_FAMILY.MONO` (`JetBrains Mono`, `Fira Code`) - used for codes, IDs, and tabular numbers.

Example:
```tsx
import { FONT_FAMILY } from "@/shared/consts/font/font";

<Box style={{ fontFamily: FONT_FAMILY.PRIMARY, fontSize: "0.875rem" }}>
  Nội dung chính
</Box>
```

## Styling Approaches in SAE-new

The project supports two primary styling mechanisms:

1. **Inline Styles / Mantine Style Props**: Best for dynamic layout configurations (e.g., responsive width, height, background transitions).
2. **CSS Modules**: Best for encapsulating reusable layouts and complex interactive components.

### 1. Custom Scrollbars

Whenever a list or panel requires scrollable content, override the default browser scrollbars with the project's minimalist styling:

```css
::-webkit-scrollbar { width: 5px; }
::-webkit-scrollbar-track { background: #F3F0EA; }
::-webkit-scrollbar-thumb { background: #C5BEB4; border-radius: 99px; }
```

### 2. Empty States and Skeletons

Always render a dedicated, branded empty state or loading loader to avoid flash-of-unstyled-content or generic text:

```tsx
import EmptyState from "@/shared/components/EmptyState";
import MissingRLVersion from "@/shared/components/svg/MissingRLVersion";
import { Loader, Box } from "@mantine/core";

// Loading State:
if (isLoading) {
  return (
    <Box style={{ display: "flex", justifyContent: "center", alignItems: "center", flex: 1 }}>
      <Loader size="md" />
    </Box>
  );
}

// Empty State:
if (!data) {
  return (
    <EmptyState
      SVG={() => <MissingRLVersion />}
      title="Chưa chọn khung điểm"
      message="Hãy chọn một khung điểm để bắt đầu chỉnh sửa"
    />
  );
}
```

## Best Practices and Premium Guidelines

1. **Avoid Hardcoded Pure Colors**: Never use pure red, green, or blue (`#ff0000`, etc.). Instead, use curated palettes, Mantine semantic colors (`blue`, `yellow`, `red`, `gray`), or soft backgrounds (`#f8f9fa`, `#F7F5F2`).
2. **Centralize Permission-Based Buttons**: Do not use vanilla Mantine `<Button>` for actionable items (like Save, Delete, Import, Export). Instead, use `<CustomButton actionType="save" />` from `@aq-fe/core-ui`, which natively implements permission checking and standardizes icons and colors.
3. **Smooth Transitions**: Always add `transition: width 0.25s ease` or similar transitions on dynamic width/height sidebars to make collapsible actions feel smooth and high-end.
4. **Consistency**: Use standard margins and paddings with Mantine variables (e.g., `p="xl"`, `p="md"`, `m="sm"`).
5. **No Placeholders**: Never leave plain text placeholders. For graphics, utilize standard custom SVGs or generate/embed images matching our professional theme.
6. **Prioritize Shared UI Reusability (`core-ui` first)**: Always use components explicitly abstracted into `@aq-fe/core-ui` rather than vanilla primitive equivalents. 
   - **CustomModal**: NEVER use vanilla Mantine `Modal`. Always use `CustomModal` from `@aq-fe/core-ui/shared/components/overlays/CustomModal`. Note that `CustomModal` uses a `disclosure` tuple instead of `opened` and `onClose` directly. Example: `<CustomModal disclosure={[opened, { open, close, toggle }]} title="...">`
   - **Inputs/Tables**: NEVER use `TextInput`, `Select`, `Button`, or `Table`. Use `CustomTextInput`, `CustomSelect`, `CustomButton`, or `CustomTanstackTable` instead. Adhering to this guarantees centralized future-proofing.
7. **Escape HTML Entities in JSX**: DO NOT use raw quotation marks (`"`) or single quotes (`'`) directly inside raw text strings within JSX tags. This triggers the Next.js/React build error: `Error: """ can be escaped with "&quot;", "&ldquo;", "&#34;", "&rdquo;". react/no-unescaped-entities`. Always substitute them with appropriate HTML entities like `&quot;`, `&ldquo;`, `&#34;`, `&rdquo;` or use template literal brackets `{"\"..."}`. 
   * **Incorrect**: `<span>Hãy chọn "Mục 1" nhé</span>`
   * **Correct**: `<span>Hãy chọn &quot;Mục 1&quot; nhé</span>`
8. **Force Production CSS Overrides via `:root` Specificity**: In Next.js production/static builds, default Mantine CSS might evaluate AFTER custom CSS modules, leading to the collapse of border-radii and custom shadows back to basic square elements. To definitively guarantee overriding precedence, always prepend the pseudo-class `:root` to all primary element selectors within custom CSS modules (e.g., rewrite `.root { ... }` as `:root .root { ... }`). This raises the specificity weight from 10 to 20, securing your custom layout perfectly against bundle reordering artifacts.
9. **Prioritize Specialized API Action Components**: When displaying standard action icons in tables or grids (like Edit or Delete), NEVER use vanilla Mantine `<ActionIcon>` combined with custom manual tooltips or icons. Instead, always use custom specialized components:
   * **For Edit/Create Modals**: `<CustomButtonCreateUpdate>` automatically adapts its visual state based on the `isUpdate` property, automatically rendering the standardized edit icon.
   * **For Deletions (CRITICAL)**: Always use `<CustomButtonDelete>` (for single row deletion) or `<CustomButtonDeleteList>` (for bulk deletion) explicitly imported from `@aq-fe/aq-core-framework/shared/components/button`. **DO NOT use any delete components from `@aq-fe/core-ui`**, as the `aq-core-framework` ones contain the correct built-in API integration logic and standardized confirmation modals.
10. **Standardize Space Gaps Between Independent Visual Blocks**: When separating distinct standalone segments inside features (e.g., the bottom of a Table and a following custom component or legend footer), always insert isolated vertical space using Mantine's `<Space h="xs" />` (approximately 10px). This provides sufficient "visual breathing air" without generating excessively wide, non-aesthetic gaps.
11. **Strict Custom Inputs Over Primitive Mantine Inputs**: NEVER import or utilize native Mantine primitive inputs directly (such as `<TextInput>`, `<Select>`, `<Textarea>`, `<Checkbox>`, `<DateInput>`, or `<DateTimePicker>`) inside feature layout components. Instead, ALWAYS import and utilize their custom abstracted wrappers from `@aq-fe/core-ui/shared/components/input` (specifically `<CustomTextInput>`, `<CustomSelect>`, `<CustomTextArea>`, `<CustomCheckbox>`, `<CustomDateInput>`, or `<CustomDateTimeInput>`). This maintains global design tokens and behavioral constraints across the entire application ecosystem.
12. **Keep Modal Titles Short and Clean**: Modal titles should remain short, clean, and concise (e.g., use "Sửa tiêu chí" or "Thêm tiêu chí" instead of long titles like "Sửa tiêu chí - Điều [Nội dung tiêu chí...]"). Keep descriptive details inside the modal inputs and labels rather than cluttering the modal header.
13. **CustomTanstackTable Column Constraints**:
   - **Always Type Columns Correctly**: You MUST type your columns array using `CustomTanstackTableColumnDef<YourType>[]` from `@aq-fe/core-ui/shared/components/dataDisplay/CustomTanstackTable/CustomTanstackTable`.
   - **Cell vs cell**: To customize the render of a cell, you MUST use the capitalized **`Cell`** property (e.g., `Cell: ({ row }) => ...`), NOT the lowercase `cell`. Using lowercase `cell` will be silently ignored.
   - **Always Use `accessorKey`, Never `id`**: When defining columns, ALWAYS use `accessorKey` instead of `id` (e.g., `accessorKey: "id"` for index). Internally, `CustomTanstackTable` maps its ID generation strictly to `accessorKey`. Using `id` without `accessorKey` results in a fatal runtime crash (`Error: Columns require an id when using a non-string header`).
   - **Actions Column**: Do not manually create an "actions" column. Instead, use the `renderRowActions={({ row }) => ...}` prop directly on the `<CustomTanstackTable>` component.
14. **Avoid Hardcoded Viewport Heights in Layouts**: Do not wrap main layout components (like tables or details views) inside a `<Box>` with `height: "100vh"` and `overflow: "hidden"` / `overflowY: "auto"`. This conflicts with the global application layout scroll and causes UI glitches (such as double scrollbars). Simply return the component directly or wrap it in a standard unconstrained container.
15. **Use readOnly instead of disabled for Business Logic Read-Only Fields**: When a form field needs to be uneditable due to business logic or a "view-only" mode (e.g., viewing details), ALWAYS use the `readOnly` prop instead of `disabled`. Using `disabled` will gray out the field and make the text harder to read, whereas `readOnly` prevents editing while keeping the text clear and readable.
16. **Top Table Actions Layout**: When using `CustomTanstackTable`, always position filters (like Select, Semester dropdowns, Checkboxes) on the LEFT side of the toolbar (near the search box) by placing them inside the `renderTopToolbarLeftActions` prop. Always position action buttons (like Export, Create, Delete, Bulk Approve) on the RIGHT side of the toolbar by placing them inside the `renderTopToolbarCustomActions` prop.
17. **Strict Enum Usage for Select/Filters**: When creating options for Dropdowns (`CustomSelect`) or checking states, NEVER hardcode magic strings/numbers (e.g., `value: "2"`). ALWAYS use the explicit backend-mapped Enums natively (e.g., `value: CadStateEnum.Pending.toString()`).
18. **Support Dark Mode Natively**: When styling components, always ensure compatibility with Dark Mode. DO NOT hardcode static colors like `backgroundColor: "white"` or `c="gray.8"`. Instead, use Mantine's built-in adaptive CSS variables (e.g., `var(--mantine-color-dark-6)`, `var(--mantine-color-gray-0)`) or the `useComputedColorScheme` hook (`isDark ? 'var(--mantine-color-blue-light)' : 'var(--mantine-color-blue-0)'`). For `Paper` and `Card` components, prefer using native props like `withBorder shadow="xs"` over custom inline styles to let Mantine automatically resolve the correct dark/light themes.
19. **Page Titles**: DO NOT use the non-existent `CustomPageTitle` component from `@aq-fe/core-ui`. For standard page headers or titles, simply use Mantine's built-in `<Title order={3}>` component instead.
20. **CustomButtonModal Children**: `<CustomButtonModal>` receives direct React nodes as its `children`. Do NOT pass a render function wrapper (e.g., `{() => (<Box>...</Box>)}`). Doing so will cause runtime render errors because the component expects valid JSX elements.
21. **CustomButtonModal in Table Row Actions**: 
    - When rendering a `<CustomButtonModal>` inside the `renderRowActions` of a table, you MUST pass the prop `isActionIcon={true}`.
    - If `isActionIcon={true}`, you MUST use **`actionIconProps`** instead of `buttonProps`. Inside `actionIconProps`, the icon component must be passed as `children` and the tooltip text inside `toolTipProps`. Example: `actionIconProps={{ color: "red", toolTipProps: { label: "Bác bỏ" }, children: <IconX size={16} /> }}`.
    - Furthermore, to prevent the action icons from stacking vertically on narrow columns, always wrap them in a `<Group wrap="nowrap">` or `<CustomFlexRow wrap="nowrap">`.
22. **CustomTabs and CustomTanstackTable Layout Spacing**: When using `<CustomTabs>` to switch between multiple `<CustomTanstackTable>` views, do NOT wrap the tables in a `<Box mt="...">` or apply margin/padding to separate them. Place the table component directly below `<CustomTabs>`. The components are already engineered to have the perfect standardized vertical rhythm. Adding arbitrary margin creates excessive whitespace that violates the design specs.
