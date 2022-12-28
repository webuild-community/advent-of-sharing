---
title: Spreading (React) props in TypeScript
author: huytd
date: 12-08-2022
---
When building a component that wrapped some other elements, it's helpful to allow the developer to forward some properties into the inner element, for example, with a `WrappedButton` component that wraps around a `button` element:

```typescript
interface WrappedButtonProps {
    className?: string;
    children: ReactNode;
}

const WrappedButton = (props: WrappedButtonProps) => {
    const { className, children } = props;
    return <button className={className}>{children}</button>
};
```

The user should be able to pass any other property into `WrappedButton` and it should be forwarded to the inner `button`:

```typescript
<WrappedButton onClick={clickHandler}>Hello</WrappedButton>
```

With the above `WrappedButtonProps` definition, we will get an error when trying to pass the `onClick` property because `onClick` is not a defined field of the `WrappedButtonProps` interface.

To bypass this, we can define the rest of the props as some unknown key values:

```typescript
interface WrappedButtonProps {
    className?: string;
    children: ReactNode;
    [key: string]: any;
}

const WrappedButton = (props: WrappedButtonProps) => {
    const { className, children, ...otherProps } = props;
    return <button className={className} {...otherProps}>{children}</button>
};
```

Keep in my, this is just a quick and dirty way to handle these kind of tasks. The right way to do it, is to extends the HTMLButtonElement or any element’s type you’re wrapping:

```typescript
interface ButtonProps extends React.ButtonHTMLAttributes<HTMLButtonElement> {
  children: ReactNode;
}

const Button: React.FC<ButtonProps> = ({ children, ...props }) => {
  return (
    <button {...props}>
      {children}
    </button>
  );
};
```

--- 

A follow up comment from **duc**:
By doing this, sometimes you'll get an editor's warning, something like this:

![image](https://user-images.githubusercontent.com/613943/206521217-3afc2a94-5db9-41d5-b5ba-ab6928c76cce.png)

The solution is:

```typescript
// another way that can be applied for all html elements
// but in some cases it would cause some warnings/errors
// interface WrappedButtonProps extends React.HTMLProps<HTMLButtonElement> {
interface WrappedButtonProps extends React.ComponentProps<'button'> {
  className?: string;
  children: React.ReactNode;
}
```

