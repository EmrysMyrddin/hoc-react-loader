# @hoc-react-loader/core
[![CircleCI](https://circleci.com/gh/alakarteio/hoc-react-loader.svg?&style=shield)](https://circleci.com/gh/alakarteio/hoc-react-loader/tree/master) [![NPM Version](https://badge.fury.io/js/hoc-react-loader.svg)](https://www.npmjs.com/package/hoc-react-loader) [![Coverage Status](https://coveralls.io/repos/github/alakarteio/hoc-react-loader/badge.svg?branch=master)](https://coveralls.io/github/alakarteio/hoc-react-loader?branch=master)

This is a [higher order component](https://facebook.github.io/react/docs/higher-order-components.html) ("HOC"). It's an advanced pattern used in React that let you reuse code logic, it can be summarized as a component factory. It improves isolation, interoperability and maintainability of your code base.

**@hoc-react-loader/core**'s purpose is to call a `load` callback passed through the `props` of a component only once (at `componentWillMount`). This is convenient to load data from a backend for instance. The component shows a loading indicator when it's waiting for the props to be defined. The loading indicator can be changed easily.

## Demos
You can test some examples [here](https://alakarteio.github.io/hoc-react-loader/).

## Installation
`yarn add @hoc-react-loader/core`

## Usage
### With `this.props`
```es6
import loader from '@hoc-react-loader/core'

const Component = ({ data }) => <div>Component {JSON.stringify(data)}</div>

export default loader({ print: ['data'] })(Component)
```
In this case, the loader waits for `this.props.data` to be truthy, then mounts its child component and calls `this.props.load` if it exists. This is useful when the parent has control over the injected data, or when the `Component` is connected with `redux`. `this.props.load` should be injected by the parent component or injected by a `Container` (redux).

The `print` parameter should be an array of props to waits. All these props should become truthy at some point.

Since the `LoadingIndicator` is not specified, `null` (nothing) is displayed while waiting for all the props. Here's an exemple with a specified loader:
```es6
import loader from '@hoc-react-loader/core'

const MyLoadingIndicator = () => <div>Waiting...</div>
const Component = ({ data }) => <div>Component {data}</div>

export default loader({ print: ['data'], LoadingIndicator: MyLoadingIndicator })(Component)
```

The `print` parameter can also be a Promise. The loading indicator is displayed until `print` Promise is resolved or rejected.

### Don't wait
```es6
import loader from '@hoc-react-loader/core'

const LoadingIndicator = () => <div>Waiting...</div>
const Component = ({ data }) => <div>Component {JSON.stringify(data)}</div>

export default loader({ LoadingIndicator })(Component)
```
In this example, the loader component doesn't wait for props. `this.props.load` is called once, but the `LoadingIndicator` component isn't displayed.

### Load as a function parameter
```es6
import loader from '@hoc-react-loader/core'

const LoadingIndicator = () => <div>Waiting...</div>
const Component = ({ data }) => <div>Component {JSON.stringify(data)}</div>

export default loader({ LoadingIndicator, load: () => console.log('here') })(Component)
```
In this case, the loader calls `this.props.load` if it exists *AND* the `load` parameter, resulting in `here` to be printed.

The default `print` parameter value is `true`. It means that in this example the `LoadingIndicator` isn't displayed.

### Load as a string parameter
```es6
import loader from '@hoc-react-loader/core'

const LoadingIndicator = () => <div>Waiting...</div>
const Component = ({ data }) => <div>Component {JSON.stringify(data)}</div>

export default loader({ LoadingIndicator, load: 'myLoader' })(Component)
```
In this case, the loader calls `this.props.myLoader` if it exists.

The default `print` parameter value is `true`. It means that in this example the `LoadingIndicator` isn't displayed.

### Print as a function
The `print` parameter can also be a function. Then the `props` and `context` are given to it (in this order), and it should return a boolean indicating wether or not the actual component should be displayed.

### Error handling
The `error` parameter allows to specify a prop that indicates wether or not a placeholder error component should be displayed in replacement of the real component.
It's usefull when data that are required for the correct display of a component are missing.

Like for the `print` prop, `error` can be a `boolean`, a `string` (referencing a prop name), an array of `string` (an array of prop names) or a `function` (whose result will be converted to `boolean`).

```js
// default error component will be displayed if 'error' prop is truthy
export default loader()(MyComponent)

// default error component will be displayed (null, meaning nothing)
export default loader({ error: true })(MyComponent)

// default error component will be displayed if 'errorMessage' prop is truthy
export default loader({ error: 'errorMessage' })(MyComponent)

// CustomErrorComponent will be displayed if 'error' prop is truthy
export default loader({ ErrorIndicator: CustomErrorComponent })(MyComponent)
```

### Delay parameter

When a component loads very quickly, you will see a flash of the loading component.
To avoid this behaviour, you can add a `delay` parameter to the loader with a time in milliseconds.
Then, the loading indicator will be rendered after the delay if the Component can't be rendered before that.

```js
// loading indicator will be displayed only after 200ms
export default loader({ print: ['data'], delay: 200 })(MyComponent)
```

By default, no delay is defined.
