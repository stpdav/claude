# React Testing Reference

## userEvent Over fireEvent

`userEvent` simulates real interaction (focus, keydown, click sequence); `fireEvent` dispatches one synthetic event:

```tsx
import userEvent from "@testing-library/user-event";

it("submits the form with typed values", async () => {
  const user = userEvent.setup();
  render(<LoginForm onSubmit={onSubmit} />);

  await user.type(screen.getByLabelText(/email/i), "alice@example.com");
  await user.click(screen.getByRole("button", { name: /log in/i }));

  expect(onSubmit).toHaveBeenCalledWith({ email: "alice@example.com" });
});
```

## Async UI

```tsx
// ✅ findBy* waits for the element to appear
expect(await screen.findByText("Loaded")).toBeInTheDocument();

// ✅ waitFor for assertions that become true later
await waitFor(() => expect(onSave).toHaveBeenCalled());

// ✅ Asserting absence after loading settles
await waitForElementToBeRemoved(() => screen.queryByText(/loading/i));
```

Never assert absence with `getBy*` - use `queryBy*`, which returns `null` instead of throwing.

## Mocking Modules

```tsx
import { vi } from "vitest";

vi.mock("@/lib/api", () => ({
  getUser: vi.fn().mockResolvedValue({ id: "1", name: "Alice" }),
}));
```

Mock at the module boundary (`lib/api`), not inside components - tests stay valid through refactors.

## Testing Custom Hooks

```tsx
import { renderHook, act } from "@testing-library/react";

it("increments the counter", () => {
  const { result } = renderHook(() => useCounter());
  act(() => result.current.increment());
  expect(result.current.count).toBe(1);
});
```

Only test hooks directly when no component exercises them - otherwise test through the component.

## Custom Render With Providers

```tsx
// test/utils.tsx - wrap once, reuse everywhere
export function renderWithProviders(ui: ReactElement) {
  return render(<ThemeProvider><SessionProvider>{ui}</SessionProvider></ThemeProvider>);
}
```

## Query Priority

1. `getByRole` (with `name`) - matches what assistive tech sees
2. `getByLabelText` - form fields
3. `getByText` - non-interactive content
4. `getByTestId` - last resort, and leave a comment justifying it
