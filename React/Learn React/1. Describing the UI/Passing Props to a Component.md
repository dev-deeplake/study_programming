> [!Learning Goals]
> 1. How to pass props to a component
>	- add them to the JSX, just like you would with HTML attributes
> 1. How to read props from a component
> 	- use destructuring syntax
> 1. How to specify default values for props
> 	- You can specify a default value like `size = 100`, which is used for missing and `undefined` props.
> 1. How to pass some JSX to a component
> 	- You can forward all props with `<Avatar {...props} />` JSX spread syntax, but don’t overuse it!
> 	- Nested JSX like `<Card><Avatar /></Card>` will appear as `Card` component’s `children` prop.
> 1. How props change over time
> 	- Props are read-only snapshots in time: every render receives a new version of props.
> 	- You can’t change props. When you need interactivity, you’ll need to set state.

![[Props]]