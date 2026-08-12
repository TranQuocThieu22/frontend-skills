---
name: app-ui-ux-standards
description: Outlines styling, UI component integration, Mantine v8 styling, custom CSS, animations, and design aesthetics across all Next.js applications in the monorepo (sae-new, srb, gam, rrs, etc.). Use when designing, styling, animating, or tweaking components in any project.
---

# UI/UX and Styling Standards Skill (All Apps)

This skill explains how to build high-fidelity, polished, premium interfaces across all Next.js applications in the monorepo using Mantine v8, custom CSS modules, global animations, and our dedicated design system (`@aq-fe/core-ui`).

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
2. **Centralize Permission-Based Buttons (Context-Based Permissions)**: Do not use vanilla Mantine `<Button>` for actionable items (like Save, Delete, Import, Export). Instead, use `<CustomButton actionType="save" />` (or `<CustomActionIcon>`) from `@aq-fe/core-ui`. These components natively implement permission checking dynamically via `PermissionContext`. **Ensure your application layout (e.g., in `sae-new`) is wrapped with the correct `PermissionProvider`** to supply the necessary backend permissions to these UI components.
3. **Smooth Transitions**: Always add `transition: width 0.25s ease` or similar transitions on dynamic width/height sidebars to make collapsible actions feel smooth and high-end.
4. **Consistency**: Use standard margins and paddings with Mantine variables (e.g., `p="xl"`, `p="md"`, `m="sm"`).
5. **No Placeholders**: Never leave plain text placeholders. For graphics, utilize standard custom SVGs or generate/embed images matching our professional theme.
6. **Prioritize Shared UI Reusability (`core-ui` first)**: Always use components explicitly abstracted into `@aq-fe/core-ui` rather than vanilla primitive equivalents. 
   - **Modals**: NEVER use vanilla Mantine `Modal` or manage modal state (`opened`/`setOpened`) manually. Always use `CustomModalTrigger` from `@aq-fe/core-ui/shared/components/overlays/CustomModalTrigger`. Example: `<CustomModalTrigger trigger={(open) => <CustomButton onClick={open}>Mở</CustomButton>}>{(close) => <Content onClose={close} />}</CustomModalTrigger>`
   - **Buttons & Icons**: NEVER use vanilla Mantine `Button` or `ActionIcon`. Always use `CustomButton` and `CustomActionIcon` from `@aq-fe/core-ui/shared/components/button/...`.
   - **Inputs/Tables**: NEVER use `TextInput`, `Select`, or `Table`. Use `CustomTextInput`, `CustomSelect`, or `CustomTanstackTable` instead. Adhering to this guarantees centralized future-proofing.
7. **Escape HTML Entities in JSX**: DO NOT use raw quotation marks (`"`) or single quotes (`'`) directly inside raw text strings within JSX tags. This triggers the Next.js/React build error: `Error: """ can be escaped with "&quot;", "&ldquo;", "&#34;", "&rdquo;". react/no-unescaped-entities`. Always substitute them with appropriate HTML entities like `&quot;`, `&ldquo;`, `&#34;`, `&rdquo;` or use template literal brackets `{"\"..."}`. 
   * **Incorrect**: `<span>Hãy chọn "Mục 1" nhé</span>`
   * **Correct**: `<span>Hãy chọn &quot;Mục 1&quot; nhé</span>`
8. **Force Production CSS Overrides via `:root` Specificity**: In Next.js production/static builds, default Mantine CSS might evaluate AFTER custom CSS modules, leading to the collapse of border-radii and custom shadows back to basic square elements. To definitively guarantee overriding precedence, always prepend the pseudo-class `:root` to all primary element selectors within custom CSS modules (e.g., rewrite `.root { ... }` as `:root .root { ... }`). This raises the specificity weight from 10 to 20, securing your custom layout perfectly against bundle reordering artifacts.
9. **Prioritize Specialized API Action Components**: When displaying standard action icons in tables or grids (like Edit or Delete), NEVER use vanilla Mantine `<ActionIcon>` combined with custom manual tooltips or icons. Instead, always use custom specialized components:
   * **For Edit/Create Modals**: `<CustomButtonCreateUpdate>` automatically adapts its visual state based on the `isUpdate` property, automatically rendering the standardized edit icon.
   * **For Deletions, API Imports, and Exports (CRITICAL)**: Always use specialized, API-integrated components (like `<CustomButtonDelete>`, `<CustomButtonDeleteList>`, `<CustomButtonImport>`, `<CustomApiSelect>`) explicitly imported from **`@aq-fe/aq-core-framework/...`**. **DO NOT use any of these components from `@aq-fe/aq-legacy-framework`**, as they are strictly isolated for the legacy backend. Core-ui provides generic visual components, but the API-bound ones for `sae-new` must come from the core framework.
10. **Standardize Space Gaps Between Independent Visual Blocks**: When separating distinct standalone segments inside features (e.g., the bottom of a Table and a following custom component or legend footer), always insert isolated vertical space using Mantine's `<Space h="xs" />` (approximately 10px). This provides sufficient "visual breathing air" without generating excessively wide, non-aesthetic gaps.
11. **Strict Custom Inputs Over Primitive Mantine Inputs**: NEVER import or utilize native Mantine primitive inputs directly (such as `<TextInput>`, `<Select>`, `<Textarea>`, `<Checkbox>`, `<DateInput>`, or `<DateTimePicker>`) inside feature layout components. Instead, ALWAYS import and utilize their custom abstracted wrappers from `@aq-fe/core-ui/shared/components/input` (specifically `<CustomTextInput>`, `<CustomSelect>`, `<CustomTextArea>`, `<CustomCheckbox>`, `<CustomDateInput>`, or `<CustomDateTimeInput>`). This maintains global design tokens and behavioral constraints across the entire application ecosystem.
12. **Keep Modal Titles Short and Clean**: Modal titles should remain short, clean, and concise (e.g., use "Sửa tiêu chí" or "Thêm tiêu chí" instead of long titles like "Sửa tiêu chí - Điều [Nội dung tiêu chí...]"). Keep descriptive details inside the modal inputs and labels rather than cluttering the modal header.
13. **CustomTanstackTable Column Constraints**:
   - **Always Type Columns Correctly**: You MUST type your columns array using `CustomTanstackTableColumnDef<YourType>[]` from `@aq-fe/core-ui/shared/components/dataDisplay/CustomTanstackTable/CustomTanstackTable`.
   - **Cell vs cell**: To customize the render of a cell, you MUST use the capitalized **`Cell`** property (e.g., `Cell: ({ row }) => ...`), NOT the lowercase `cell`. Using lowercase `cell` will be silently ignored.
   - **Always Use `accessorKey`, Never `id`**: When defining columns, ALWAYS use `accessorKey` instead of `id` (e.g., `accessorKey: "id"` for index). Internally, `CustomTanstackTable` maps its ID generation strictly to `accessorKey`. Using `id` without `accessorKey` results in a fatal runtime crash (`Error: Columns require an id when using a non-string header`).
   - **unsupportedTooltip (CRITICAL)**: When a column is desired on the UI but the backend API has not yet supported/returned it (e.g., missing properties), you MUST set `unsupportedTooltip: "Backend chưa hỗ trợ trường này"` (or a similar descriptive label) on that column definition. This automatically renders an info icon and tooltip next to the header name to alert users and developers.
   - **Actions Column**: Do not manually create an "actions" column. Instead, use the `renderRowActions={({ row }) => ...}` prop directly on the `<CustomTanstackTable>` component.
   - **Actions Column Size**: Do not set a fixed `size` (e.g. `size: 100`) for the "Thao tác" (actions) column. Let it auto-size based on content so action buttons are not squished or constrained.
14. **Avoid Hardcoded Viewport Heights and Redundant Padding in Layouts**: Do not wrap main layout components (like tables or details views) inside a `<Box>` or `<div>` with `height: "100vh"`, `padding: 16`, and `overflow: "hidden"` / `overflowY: "auto"`. The `CustomAppShell` already provides default structural padding and scroll handling. Wrapping components to add manual padding or height conflicts with the global layout and causes UI glitches (such as double scrollbars or clipped content). Simply return the layout grid or stack directly.
15. **Use readOnly instead of disabled for Business Logic Read-Only Fields**: When a form field needs to be uneditable due to business logic or a "view-only" mode (e.g., viewing details), ALWAYS use the `readOnly` prop instead of `disabled`. Using `disabled` will gray out the field and make the text harder to read, whereas `readOnly` prevents editing while keeping the text clear and readable.
16. **Top Table Actions Layout**: When using `CustomTanstackTable`, always position filters (like Select, Semester dropdowns, Checkboxes) on the LEFT side of the toolbar (near the search box) by placing them inside the `renderTopToolbarLeftActions` prop. Always position action buttons (like Export, Create, Delete, Bulk Approve) on the RIGHT side of the toolbar by placing them inside the `renderTopToolbarCustomActions` prop.
17. **Strict Enum Usage for Select/Filters**: When creating options for Dropdowns (`CustomSelect`) or checking states, NEVER hardcode magic strings/numbers (e.g., `value: "2"`). ALWAYS use the explicit backend-mapped Enums natively (e.g., `value: CadStateEnum.Pending.toString()`).
18. **Support Dark Mode Natively**: When styling components, always ensure compatibility with Dark Mode. DO NOT hardcode static colors like `backgroundColor: "white"` or `c="gray.8"`. Instead, use Mantine's built-in adaptive CSS variables (e.g., `var(--mantine-color-dark-6)`, `var(--mantine-color-gray-0)`) or the `useComputedColorScheme` hook (`isDark ? 'var(--mantine-color-blue-light)' : 'var(--mantine-color-blue-0)'`). For `Paper` and `Card` components, prefer using native props like `withBorder shadow="xs"` over custom inline styles to let Mantine automatically resolve the correct dark/light themes.
19. **Page Titles**: DO NOT use the non-existent `CustomPageTitle` component from `@aq-fe/core-ui`. For standard page headers or titles, simply use Mantine's built-in `<Title order={3}>` component instead.
20. **CustomModalTrigger Children**: `<CustomModalTrigger>` uses the render props pattern.
    - **Trigger**: The `trigger` prop is a function `(open) => ReactNode`. Use this to conditionally render a `<CustomButton onClick={open}>` or `<CustomActionIcon onClick={open}>` depending on whether it's used in a top toolbar or a table row action.
    - **Children**: The `children` prop can be a function `(close) => ReactNode` to allow you to manually close the modal (e.g. `onClick={close}`) without needing to manage the disclosure state yourself.
21. **CustomModalTrigger in Table Row Actions**: 
    - When rendering a `<CustomModalTrigger>` inside the `renderRowActions` of a table, simply use the `trigger` prop to return an `<ActionIcon>`.
    - To prevent the action icons from stacking vertically on narrow columns, always wrap them in a `<Group wrap="nowrap">` or `<CustomFlexRow wrap="nowrap">`.
22. **CustomTabs and CustomTanstackTable Layout Spacing**: When using `<CustomTabs>` to switch between multiple `<CustomTanstackTable>` views, do NOT wrap the tables in a `<Box mt="...">` or apply margin/padding to separate them. Place the table component directly below `<CustomTabs>`. The components are already engineered to have the perfect standardized vertical rhythm. Adding arbitrary margin creates excessive whitespace that violates the design specs.
23. **CustomButton Action Types**: When implementing action buttons using `<CustomButton>` for destructive actions (e.g. deleting, canceling, rejecting, or removing data), you MUST use `actionType="delete"` (instead of "save") to ensure correct standardized semantic coloring and icons.
24. **Confirm/Delete Modals**: Do not use `modals.openConfirmModal` from `@mantine/modals` to trigger confirm dialogues. Instead, define and control standard `<Modal>` components locally within your feature to handle confirmation logic explicitly.
25. **Modal Footer Actions**: When creating action buttons at the bottom of a Modal (Footer), ALWAYS align them to the right using `<Group justify="flex-end">`. 
    - The dismissive action (e.g. "Đóng", "Hủy") MUST be placed on the left side of the group with `variant="default"` (gray outline).
    - The primary action (e.g. "Lưu", "Xóa") MUST be placed on the right side of the group with a filled color (e.g., `color="blue"` for saving, `color="red"` for deleting).
    - Do NOT use `<Group justify="center">` for modal footers.
26. **Button and ActionIcon Variants**: DO NOT use `variant="subtle"` or `variant="outline"`. For standard `Button` components, use `variant="filled"` or omit the variant prop completely (as `filled` is the default). For `ActionIcon` components, ALWAYS use `variant="light"`.
