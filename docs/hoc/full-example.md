---
id: full_example
sidebar_label: Full HOC Example
title: Section 0: Full HOC Example
---

> This is an HOC example for you to copy and paste. If you certain pieces don't make sense for you, head to [Section 1](#section-1-react-hoc-docs-in-typescript) to get a detailed walkthrough via a complete translation of the React docs in TypeScript.

Sometimes you want a simple way to inject props from somewhere else (either a global store or a provider) and don't want to continually pass down the props for it. Context is great for it, but then the values from the context can only be used in your `render` function. A HoC will provide these values as props.

**The injected props**

```ts
interface WithThemeProps {
  primaryColor: string;
}
```

**Usage in the component**

The goal is to have the props available on the interface for the component, but subtracted out for the consumers of the component when wrapped in the HoC.

```ts
interface Props extends WithThemeProps {
  children: React.ReactNode;
}

class MyButton extends React.Component<Props> {
  public render() {
    // Render an the element using the theme and other props.
  }

  private someInternalMethod() {
    // The theme values are also available as props here.
  }
}

export default withTheme(MyButton);
```

**Consuming the Component**

Now when consuming the component you can omit the `primaryColor` prop or override the one provided through context.

```tsx
<MyButton>Hello button</MyButton> // Valid
<MyButton primaryColor="#333">Hello Button</MyButton> // Also valid
```

**Declaring the HoC**

The actual HoC.

```tsx
export function withTheme<T extends WithThemeProps = WithThemeProps>(
  WrappedComponent: React.ComponentType<T>
) {
  // Try to create a nice displayName for React Dev Tools.
  const displayName =
    WrappedComponent.displayName || WrappedComponent.name || "Component";

  // Creating the inner component. The calculated Props type here is the where the magic happens.
  return class ComponentWithTheme extends React.Component<
    Optionalize<T, WithThemeProps>
  > {
    public static displayName = `withTheme(${displayName})`;

    public render() {
      // Fetch the props you want inject. This could be done with context instead.
      const themeProps = getThemePropsFromSomeWhere();

      // this.props comes afterwards so the can override the default ones.
      return <WrappedComponent {...themeProps} {...(this.props as T)} />;
    }
  };
}
```

Note that the `{...this.props as T}` assertion is needed because of a current bug in TS 3.2 https://github.com/Microsoft/TypeScript/issues/28938#issuecomment-450636046

For `Optionalize` details check out the [utility types section](https://github.com/typescript-cheatsheets/typescript-utilities-cheatsheet#utility-types).

Here is a more advanced example of a dynamic higher order component that bases some of its parameters on the props of the component being passed in:

```tsx
// inject static values to a component so that they're always provided
export function inject<TProps, TInjectedKeys extends keyof TProps>(
  Component: React.JSXElementConstructor<TProps>,
  injector: Pick<TProps, TInjectedKeys>
) {
  return function Injected(props: Omit<TProps, TInjectedKeys>) {
    return <Component {...(props as TProps)} {...injector} />;
  };
}
```

### Using `forwardRef`

For "true" reusability you should also consider exposing a ref for your HOC. You can use `React.forwardRef<Ref, Props>` as documented in [the basic cheatsheet](https://github.com/typescript-cheatsheets/react-typescript-cheatsheet/blob/master/README.md#forwardrefcreateref), but we are interested in more real world examples. [Here is a nice example in practice](https://gist.github.com/OliverJAsh/d2f462b03b3e6c24f5588ca7915d010e) from @OliverJAsh.