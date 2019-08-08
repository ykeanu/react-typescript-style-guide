# react-typescript-style-guide


## React & Redux in TypeScript - Static Typing Guide



**React, Typescript Ecosystem**

- [typesafe-actions](https://github.com/piotrwitek/typesafe-actions) - Typesafe utilities for "action-creators" in Redux / Flux Architecture  
- [utility-types](https://github.com/piotrwitek/utility-types) - Collection of generic types for TypeScript, complementing built-in mapped types and aliases - think lodash for reusable types.  
- [react-redux-typescript-scripts](https://github.com/piotrwitek/react-redux-typescript-scripts) - dev-tools configuration files shared between projects based on this guide  


🌟 - _New or updated section_

## Table of Contents
- [Example](#exa,[;e)


---

# Installation

### Type-Definitions for React & Redux
```
npm i -D @types/react @types/react-dom @types/react-redux
```

"react" - `@types/react`  
"react-dom" - `@types/react-dom`  
"redux" - (types included with npm package)*  
"react-redux" - `@types/react-redux`  

> *NB: Guide is based on types for Redux >= v4.x.x. To make it work with Redux v3.x.x please refer to [this config](https://github.com/piotrwitek/react-redux-typescript-guide/blob/master/playground/tsconfig.json#L5))  

[⇧ back to top](#table-of-contents)

---

# React - Type-Definitions Cheatsheet

#### `React.FC<Props>` | `React.FunctionComponent<Props>`
Type representing a functional component
```tsx
const MyComponent: React.FC<Props> = ...
```

#### `React.Component<Props, State>`
Type representing a class component
```tsx
class MyComponent extends React.Component<Props, State> { ...
```

#### `React.ComponentType<Props>`
Type representing union of (React.FC<Props> | React.Component<Props>) - used in HOC
```tsx
const withState = <P extends WrappedComponentProps>(
  WrappedComponent: React.ComponentType<P>,
) => { ...
```

#### `React.ComponentProps<typeof XXX>`
Gets Props type of a specified component XXX (WARNING: does not work with statically declared default props and generic props)
```tsx
type MyComponentProps = React.ComponentProps<typeof MyComponent>;
```

#### `React.ReactElement` | `JSX.Element`
Type representing a concept of React Element - representation of a native DOM component (e.g. `<div />`), or a user-defined composite component (e.g. `<MyComponent />`)
```tsx
const elementOnly: React.ReactElement = <div /> || <MyComponent />;
```

#### `React.ReactNode`
Type representing any possible type of React node (basically ReactElement (including Fragments and Portals) + primitive JS types)
```tsx
const elementOrPrimitive: React.ReactNode = 'string' || 0 || false || null || undefined || <div /> || <MyComponent />;
const Component = ({ children: React.ReactNode }) => ...
```

#### `React.CSSProperties`
Type representing style object in JSX - for css-in-js styles
```tsx
const styles: React.CSSProperties = { flexDirection: 'row', ...
const element = <div style={styles} ...
```

#### `React.HTMLProps<HTMLXXXElement>`
Type representing Props of specified HTML Element - for extending HTML Elements
```tsx
const Input: React.FC<Props & React.HTMLProps<HTMLInputElement>> = props => { ... }

<Input about={...} accept={...} alt={...} ... />
```

#### `React.ReactEventHandler<HTMLXXXElement>`
Type representing generic event handler - for declaring event handlers
```tsx
const handleChange: React.ReactEventHandler<HTMLInputElement> = (ev) => { ... } 

<input onChange={handleChange} ... />
```

#### `React.XXXEvent<HTMLXXXElement>`
Type representing more specific event handler. Some common event examples: `ChangeEvent, FormEvent, FocusEvent, KeyboardEvent, MouseEvent, DragEvent, PointerEvent, WheelEvent, TouchEvent`.
```tsx
const handleChange = (ev: React.MouseEvent<HTMLDivElement>) => { ... }

<div onMouseMove={handleChange} ... />
```

[⇧ back to top](#table-of-contents)

---

# React - Typing Patterns

## Function Components - FC

#### - FC counter

```tsx
import * as React from 'react';

type Props = {
  label: string;
  count: number;
  onIncrement: () => void;
};

export const FCCounter: React.FC<Props> = props => {
  const { label, count, onIncrement } = props;

  const handleIncrement = () => {
    onIncrement();
  };

  return (
    <div>
      <span>
        {label}: {count}
      </span>
      <button type="button" onClick={handleIncrement}>
        {`Increment`}
      </button>
    </div>
  );
};

```

[⟩⟩⟩ demo](https://piotrwitek.github.io/react-redux-typescript-guide/#fccounter)

[⇧ back to top](#table-of-contents)

#### - spread attributes [link](https://facebook.github.io/react/docs/jsx-in-depth.html#spread-attributes)

```tsx
import * as React from 'react';

type Props = {
  className?: string;
  style?: React.CSSProperties;
};

export const FCSpreadAttributes: React.FC<Props> = props => {
  const { children, ...restProps } = props;

  return <div {...restProps}>{children}</div>;
};

```

[⟩⟩⟩ demo](https://piotrwitek.github.io/react-redux-typescript-guide/#fcspreadattributes)

[⇧ back to top](#table-of-contents)

---

## Class Components

#### - class counter

```tsx
import * as React from 'react';

type Props = {
  label: string;
};

type State = {
  count: number;
};

export class ClassCounter extends React.Component<Props, State> {
  readonly state: State = {
    count: 0,
  };

  handleIncrement = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    const { handleIncrement } = this;
    const { label } = this.props;
    const { count } = this.state;

    return (
      <div>
        <span>
          {label}: {count}
        </span>
        <button type="button" onClick={handleIncrement}>
          {`Increment`}
        </button>
      </div>
    );
  }
}

```

[⟩⟩⟩ demo](https://piotrwitek.github.io/react-redux-typescript-guide/#classcounter)

[⇧ back to top](#table-of-contents)

#### - with default props

```tsx
import * as React from 'react';

type Props = {
  label: string;
  initialCount: number;
};

type State = {
  count: number;
};

export class ClassCounterWithDefaultProps extends React.Component<
  Props,
  State
> {
  static defaultProps = {
    initialCount: 0,
  };

  readonly state: State = {
    count: this.props.initialCount,
  };

  componentWillReceiveProps({ initialCount }: Props) {
    if (initialCount != null && initialCount !== this.props.initialCount) {
      this.setState({ count: initialCount });
    }
  }

  handleIncrement = () => {
    this.setState({ count: this.state.count + 1 });
  };

  render() {
    const { handleIncrement } = this;
    const { label } = this.props;
    const { count } = this.state;

    return (
      <div>
        <span>
          {label}: {count}
        </span>
        <button type="button" onClick={handleIncrement}>
          {`Increment`}
        </button>
      </div>
    );
  }
}

```

[⟩⟩⟩ demo](https://piotrwitek.github.io/react-redux-typescript-guide/#classcounterwithdefaultprops)

[⇧ back to top](#table-of-contents)

---

## Generic Components
- easily create typed component variations and reuse common logic
- common use case is a generic list components

#### - generic list

```tsx
import * as React from 'react';

export interface GenericListProps<T> {
  items: T[];
  itemRenderer: (item: T) => JSX.Element;
}

export class GenericList<T> extends React.Component<GenericListProps<T>, {}> {
  render() {
    const { items, itemRenderer } = this.props;

    return (
      <div>
        {items.map(itemRenderer)}
      </div>
    );
  }
}

```

[⟩⟩⟩ demo](https://piotrwitek.github.io/react-redux-typescript-guide/#genericlist)

[⇧ back to top](#table-of-contents)

---

## Render Props
> https://reactjs.org/docs/render-props.html

#### - name provider
> simple component using children as a render prop

```tsx
import * as React from 'react';

interface NameProviderProps {
  children: (state: NameProviderState) => React.ReactNode;
}

interface NameProviderState {
  readonly name: string;
}

export class NameProvider extends React.Component<NameProviderProps, NameProviderState> {
  readonly state: NameProviderState = { name: 'Piotr' };

  render() {
    return this.props.children(this.state);
  }
}

```

[⟩⟩⟩ demo](https://piotrwitek.github.io/react-redux-typescript-guide/#nameprovider)

[⇧ back to top](#table-of-contents)

#### - mouse provider
> `Mouse` component found in [Render Props React Docs](https://reactjs.org/docs/render-props.html#use-render-props-for-cross-cutting-concerns)

```tsx
import * as React from 'react';

export interface MouseProviderProps {
  render: (state: MouseProviderState) => React.ReactNode;
}

interface MouseProviderState {
  readonly x: number;
  readonly y: number;
}

export class MouseProvider extends React.Component<MouseProviderProps, MouseProviderState> {
  readonly state: MouseProviderState = { x: 0, y: 0 };

  handleMouseMove = (event: React.MouseEvent<HTMLDivElement>) => {
    this.setState({
      x: event.clientX,
      y: event.clientY,
    });
  };

  render() {
    return (
      <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>
        {/*
          Instead of providing a static representation of what <Mouse> renders,
          use the `render` prop to dynamically determine what to render.
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

```

[⟩⟩⟩ demo](https://piotrwitek.github.io/react-redux-typescript-guide/#mouseprovider)

[⇧ back to top](#table-of-contents)

---

## Higher-Order Components
> https://reactjs.org/docs/higher-order-components.html

#### - withState
Adds state to a stateless counter

```tsx
import * as React from 'react';
import { Subtract } from 'utility-types';

// These props will be subtracted from base component props
interface InjectedProps {
  count: number;
  onIncrement: () => void;
}

export const withState = <BaseProps extends InjectedProps>(
  _BaseComponent: React.ComponentType<BaseProps>
) => {
  // fix for TypeScript issues: https://github.com/piotrwitek/react-redux-typescript-guide/issues/111
  const BaseComponent = _BaseComponent as React.ComponentType<InjectedProps>;

  type HocProps = Subtract<BaseProps, InjectedProps> & {
    // here you can extend hoc with new props
    initialCount?: number;
  };
  type HocState = {
    readonly count: number;
  };

  return class Hoc extends React.Component<HocProps, HocState> {
    // Enhance component name for debugging and React-Dev-Tools
    static displayName = `withState(${BaseComponent.name})`;
    // reference to original wrapped component
    static readonly WrappedComponent = BaseComponent;

    readonly state: HocState = {
      count: Number(this.props.initialCount) || 0,
    };

    handleIncrement = () => {
      this.setState({ count: this.state.count + 1 });
    };

    render() {
      const { ...restProps } = this.props;
      const { count } = this.state;

      return (
        <BaseComponent
          count={count} // injected
          onIncrement={this.handleIncrement} // injected
          {...restProps}
        />
      );
    }
  };
};

```
<details><summary><i>Click to expand</i></summary><p>

```tsx
import * as React from 'react';

import { withState } from '../hoc';
import { FCCounter } from '../components';

const FCCounterWithState = withState(FCCounter);

export default () => <FCCounterWithState label={'FCCounterWithState'} />;

```
</p></details>

[⇧ back to top](#table-of-contents)

#### - withErrorBoundary
Adds error handling using componentDidCatch to any component

```tsx
import * as React from 'react';
import { Subtract } from 'utility-types';

const MISSING_ERROR = 'Error was swallowed during propagation.';

// These props will be subtracted from base component props
interface InjectedProps {
  onReset: () => void;
}

export const withErrorBoundary = <BaseProps extends InjectedProps>(
  _BaseComponent: React.ComponentType<BaseProps>
) => {
  // fix for TypeScript issues: https://github.com/piotrwitek/react-redux-typescript-guide/issues/111
  const BaseComponent = _BaseComponent as React.ComponentType<InjectedProps>;

  type HocProps = Subtract<BaseProps, InjectedProps> & {
    // here you can extend hoc with new props
  };
  type HocState = {
    readonly error: Error | null | undefined;
  };

  return class Hoc extends React.Component<HocProps, HocState> {
    // Enhance component name for debugging and React-Dev-Tools
    static displayName = `withErrorBoundary(${BaseComponent.name})`;
    // reference to original wrapped component
    static readonly WrappedComponent = BaseComponent;

    readonly state: HocState = {
      error: undefined,
    };

    componentDidCatch(error: Error | null, info: object) {
      this.setState({ error: error || new Error(MISSING_ERROR) });
      this.logErrorToCloud(error, info);
    }

    logErrorToCloud = (error: Error | null, info: object) => {
      // TODO: send error report to service provider
    };

    handleReset = () => {
      this.setState({ error: undefined });
    };

    render() {
      const { children, ...restProps } = this.props;
      const { error } = this.state;

      if (error) {
        return (
          <BaseComponent
            onReset={this.handleReset} // injected
            {...restProps}
          />
        );
      }

      return children;
    }
  };
};

```
<details><summary><i>Click to expand</i></summary><p>

```tsx
import React, {useState} from 'react';

import { withErrorBoundary } from '../hoc';
import { ErrorMessage } from '../components';

const ErrorMessageWithErrorBoundary =
  withErrorBoundary(ErrorMessage);

const BrokenComponent = () => {
  throw new Error('I\'m broken! Don\'t render me.');
};

const BrokenButton = () => {
  const [shouldRenderBrokenComponent, setShouldRenderBrokenComponent] =
    useState(false);

  if (shouldRenderBrokenComponent) {
    return <BrokenComponent />;
  }

  return (
    <button
      type="button"
      onClick={() => {
        setShouldRenderBrokenComponent(true);
      }}
    >
      {`Throw nasty error`}
    </button>
  );
};

export default () => (
  <ErrorMessageWithErrorBoundary>
    <BrokenButton />
  </ErrorMessageWithErrorBoundary>
);

```
</p></details>

[⇧ back to top](#table-of-contents)

---

## Redux Connected Components

#### - redux connected counter

```tsx
import Types from 'MyTypes';
import { connect } from 'react-redux';

import { countersActions, countersSelectors } from '../features/counters';
import { FCCounter } from '../components';

const mapStateToProps = (state: Types.RootState) => ({
  count: countersSelectors.getReduxCounter(state.counters),
});

const dispatchProps = {
  onIncrement: countersActions.increment,
};

export const FCCounterConnected = connect(
  mapStateToProps,
  dispatchProps
)(FCCounter);

```
<details><summary><i>Click to expand</i></summary><p>

```tsx
import * as React from 'react';

import { FCCounterConnected } from '.';

export default () => <FCCounterConnected label={'FCCounterConnected'} />;

```
</p></details>

[⇧ back to top](#table-of-contents)

#### - redux connected counter with own props

```tsx
import Types from 'MyTypes';
import { connect } from 'react-redux';

import { countersActions, countersSelectors } from '../features/counters';
import { FCCounter } from '../components';

type OwnProps = {
  initialCount?: number;
};

const mapStateToProps = (state: Types.RootState, ownProps: OwnProps) => ({
  count:
    countersSelectors.getReduxCounter(state.counters) +
    (ownProps.initialCount || 0),
});

const dispatchProps = {
  onIncrement: countersActions.increment,
};

export const FCCounterConnectedOwnProps = connect(
  mapStateToProps,
  dispatchProps
)(FCCounter);

```
<details><summary><i>Click to expand</i></summary><p>

```tsx
import * as React from 'react';

import { FCCounterConnectedOwnProps } from '.';

export default () => (
  <FCCounterConnectedOwnProps
    label={'FCCounterConnectedOwnProps'}
    initialCount={10}
  />
);

```
</p></details>

[⇧ back to top](#table-of-contents)

#### - redux connected counter with `redux-thunk` integration

```tsx
import Types from 'MyTypes';
import { bindActionCreators, Dispatch } from 'redux';
import { connect } from 'react-redux';
import * as React from 'react';

import { countersActions } from '../features/counters';

// Thunk Action
const incrementWithDelay = () => async (dispatch: Dispatch): Promise<void> => {
  setTimeout(() => dispatch(countersActions.increment()), 1000);
};

const mapStateToProps = (state: Types.RootState) => ({
  count: state.counters.reduxCounter,
});

const mapDispatchToProps = (dispatch: Dispatch<Types.RootAction>) =>
  bindActionCreators(
    {
      onIncrement: incrementWithDelay,
    },
    dispatch
  );

type Props = ReturnType<typeof mapStateToProps> &
  ReturnType<typeof mapDispatchToProps> & {
    label: string;
  };

export const FCCounter: React.FC<Props> = props => {
  const { label, count, onIncrement } = props;

  const handleIncrement = () => {
    // Thunk action is correctly typed as promise
    onIncrement().then(() => {
      // ...
    });
  };

  return (
    <div>
      <span>
        {label}: {count}
      </span>
      <button type="button" onClick={handleIncrement}>
        {`Increment`}
      </button>
    </div>
  );
};

export const FCCounterConnectedBindActionCreators = connect(
  mapStateToProps,
  mapDispatchToProps
)(FCCounter);

```
<details><summary><i>Click to expand</i></summary><p>

```tsx
import * as React from 'react';

import { FCCounterConnectedBindActionCreators } from '.';

export default () => (
  <FCCounterConnectedBindActionCreators
    label={'FCCounterConnectedBindActionCreators'}
  />
);

```
</p></details>

[⇧ back to top](#table-of-contents)

## Context

> https://reactjs.org/docs/context.html

#### ThemeContext

```tsx
import * as React from 'react';

export type Theme = React.CSSProperties;

type Themes = {
  dark: Theme;
  light: Theme;
};

export const themes: Themes = {
  dark: {
    color: 'black',
    backgroundColor: 'white',
  },
  light: {
    color: 'white',
    backgroundColor: 'black',
  },
};

export type ThemeContextProps = { theme: Theme; toggleTheme?: () => void };
const ThemeContext = React.createContext<ThemeContextProps>({ theme: themes.light });

export default ThemeContext;

```

[⇧ back to top](#table-of-contents)

#### ThemeProvider

```tsx
import React from 'react';
import ThemeContext, { themes, Theme } from './theme-context';
import ToggleThemeButton from './theme-consumer';

interface State {
  theme: Theme;
}
export class ThemeProvider extends React.Component<{}, State> {
  readonly state: State = { theme: themes.light };

  toggleTheme = () => {
    this.setState(state => ({
      theme: state.theme === themes.light ? themes.dark : themes.light,
    }));
  }

  render() {
    const { theme } = this.state;
    const { toggleTheme } = this;
    return (
      <ThemeContext.Provider value={{ theme, toggleTheme }}>
        <ToggleThemeButton />
      </ThemeContext.Provider>
    );
  }
}

```

[⇧ back to top](#table-of-contents)

#### ThemeConsumer

```tsx
import * as React from 'react';
import ThemeContext from './theme-context';

type Props = {};

export default function ToggleThemeButton(props: Props) {
  return (
    <ThemeContext.Consumer>
      {({ theme, toggleTheme }) => <button style={theme} onClick={toggleTheme} {...props} />}
    </ThemeContext.Consumer>
  );
}

```

#### ThemeConsumer in class component

```tsx
import * as React from 'react';
import ThemeContext from './theme-context';

type Props = {};

export class ToggleThemeButtonClass extends React.Component<Props> {
  static contextType = ThemeContext;
  context!: React.ContextType<typeof ThemeContext>;

  render() {
    const { theme, toggleTheme } = this.context;
    return (
      <button style={theme} onClick={toggleTheme}>
        Toggle Theme
      </button>
    );
  }
}

```

[Implementation with Hooks](#--usecontext)

[⇧ back to top](#table-of-contents)

## Hooks

> https://reactjs.org/docs/hooks-intro.html

#### - useState

> https://reactjs.org/docs/hooks-reference.html#usestate

```tsx
import * as React from 'react';

type Props = { initialCount: number };

export default function Counter({initialCount}: Props) {
  const [count, setCount] = React.useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
    </>
  );
}

```

[⇧ back to top](#table-of-contents)

#### - useReducer
Hook for state management like Redux in a function component.

```tsx
import * as React from 'react';

interface State {
  count: number;
}

type Action = { type: 'reset' } | { type: 'increment' } | { type: 'decrement' };

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      throw new Error();
  }
}

interface CounterProps {
  initialCount: number;
}

function Counter({ initialCount }: CounterProps) {
  const [state, dispatch] = React.useReducer(reducer, {
    count: initialCount,
  });

  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
    </>
  );
}

export default Counter;

```

[⇧ back to top](#table-of-contents)

#### - useContext

> https://reactjs.org/docs/hooks-reference.html#usecontext

```tsx
import * as React from 'react';
import ThemeContext from '../context/theme-context';

type Props = {};

export default function ThemeToggleButton(props: Props) {
  const { theme, toggleTheme } = React.useContext(ThemeContext);
  return (
    <button onClick={toggleTheme} style={theme} >
      Toggle Theme
    </button>
  );
}

```

[⇧ back to top](#table-of-contents)

---

# Redux - Typing Patterns

## Store Configuration

### Create Global RootState and RootAction Types

#### `RootState` - type representing root state-tree
Can be imported in connected components to provide type-safety to Redux `connect` function

#### `RootAction` - type representing union type of all action objects
Can be imported in various layers receiving or sending redux actions like: reducers, sagas or redux-observables epics

```tsx
import { StateType, ActionType } from 'typesafe-actions';

declare module 'MyTypes' {
  export type Store = StateType<typeof import('./index').default>;
  export type RootAction = ActionType<typeof import('./root-action').default>;
  export type RootState = StateType<typeof import('./root-reducer').default>;
}

declare module 'typesafe-actions' {
  interface Types {
    RootAction: ActionType<typeof import('./root-action').default>;
  }
}

```

[⇧ back to top](#table-of-contents)

### Create Store

When creating a store instance we don't need to provide any additional types. It will set-up a **type-safe Store instance** using type inference.
> The resulting store instance methods like `getState` or `dispatch` will be type checked and will expose all type errors

```tsx
import { RootAction, RootState, Services } from 'MyTypes';
import { createStore, applyMiddleware } from 'redux';
import { createEpicMiddleware } from 'redux-observable';

import { composeEnhancers } from './utils';
import rootReducer from './root-reducer';
import rootEpic from './root-epic';
import services from '../services';

export const epicMiddleware = createEpicMiddleware<
  RootAction,
  RootAction,
  RootState,
  Services
>({
  dependencies: services,
});

// configure middlewares
const middlewares = [epicMiddleware];
// compose enhancers
const enhancer = composeEnhancers(applyMiddleware(...middlewares));

// rehydrate state on app start
const initialState = {};

// create store
const store = createStore(rootReducer, initialState, enhancer);

epicMiddleware.run(rootEpic);

// export store singleton instance
export default store;

```

---

## Action Creators

> We'll be using a battle-tested library [![NPM Downloads](https://img.shields.io/npm/dm/typesafe-actions.svg)](https://www.npmjs.com/package/typesafe-actions)
 that'll help retain complete type soundness and simplify maintenace of **types in Redux Architectures** [`typesafe-actions`](https://github.com/piotrwitek/typesafe-actions#typesafe-actions)

> You can find more real-world examples and in-depth tutorial in: [Typesafe-Actions - Tutorial](https://github.com/piotrwitek/typesafe-actions#tutorial)!

A solution below is using a simple factory function to automate the creation of type-safe action creators. The goal is to decrease maintenance effort and reduce code repetition of type annotations for actions and creators. The result is completely typesafe action-creators and their actions.

```tsx
import { action } from 'typesafe-actions';

import { ADD, INCREMENT } from './constants';

// CLASSIC API
export const increment = () => action(INCREMENT);
export const add = (amount: number) => action(ADD, amount);

// ALTERNATIVE API - allow to use reference to "action-creator" function instead of "type constant"
// e.g. case getType(increment): return { ... }
// This will allow to completely eliminate need for "constants" in your application, more info here:
// https://github.com/piotrwitek/typesafe-actions#behold-the-mighty-tutorial

// OPTION 1 (with generics):
// import { createStandardAction } from 'typesafe-actions';
// export const increment = createStandardAction(INCREMENT)<void>();
// export const add = createStandardAction(ADD)<number>();

// OPTION 2 (with resolve callback):
// import { createAction } from 'typesafe-actions';
// export const increment = createAction(INCREMENT);
// export const add = createAction(ADD, resolve => {
//   return (amount: number) => resolve(amount);
// });

```
<details><summary><i>Click to expand</i></summary><p>

```tsx
import store from '../../store';
import { countersActions as counter } from '../counters';

// store.dispatch(counter.increment(1)); // Error: Expected 0 arguments, but got 1.
store.dispatch(counter.increment()); // OK

// store.dispatch(counter.add()); // Error: Expected 1 arguments, but got 0.
store.dispatch(counter.add(1)); // OK

```
</p></details>

[⇧ back to top](#table-of-contents)

---

## Reducers


[⇧ back to top](#table-of-contents)

### Typing reducer

> to understand following section make sure to learn about [Type Inference](https://www.typescriptlang.org/docs/handbook/type-inference.html), [Control flow analysis](https://github.com/Microsoft/TypeScript/wiki/What%27s-new-in-TypeScript#control-flow-based-type-analysis) and [Tagged union types](https://github.com/Microsoft/TypeScript/wiki/What%27s-new-in-TypeScript#tagged-union-types)

```tsx
import { combineReducers } from 'redux';
import { ActionType } from 'typesafe-actions';

import { Todo, TodosFilter } from './models';
import * as actions from './actions';
import { ADD, CHANGE_FILTER, TOGGLE } from './constants';

export type TodosAction = ActionType<typeof actions>;

export type TodosState = Readonly<{
  todos: Todo[];
  todosFilter: TodosFilter;
}>;
const initialState: TodosState = {
  todos: [],
  todosFilter: TodosFilter.All,
};

export default combineReducers<TodosState, TodosAction>({
  todos: (state = initialState.todos, action) => {
    switch (action.type) {
      case ADD:
        return [...state, action.payload];

      case TOGGLE:
        return state.map(item =>
          item.id === action.payload
            ? { ...item, completed: !item.completed }
            : item
        );

      default:
        return state;
    }
  },
  todosFilter: (state = initialState.todosFilter, action) => {
    switch (action.type) {
      case CHANGE_FILTER:
        return action.payload;

      default:
        return state;
    }
  },
});

```

[⇧ back to top](#table-of-contents)

### Typing reducer with `typesafe-actions`
> Notice we are not required to use any generic type parameter in the API. Try to compare it with regular reducer as they are equivalent.

```tsx
import { combineReducers } from 'redux';
import { createReducer } from 'typesafe-actions';

import { Todo, TodosFilter } from './models';
import { ADD, CHANGE_FILTER, TOGGLE } from './constants';

export type TodosState = Readonly<{
  todos: Todo[];
  todosFilter: TodosFilter;
}>;
const initialState: TodosState = {
  todos: [],
  todosFilter: TodosFilter.All,
};

const todos = createReducer(initialState.todos)
  .handleAction(ADD, (state, action) => [...state, action.payload])
  .handleAction(TOGGLE, (state, action) =>
    state.map(item =>
      item.id === action.payload
        ? { ...item, completed: !item.completed }
        : item
    )
  );

const todosFilter = createReducer(initialState.todosFilter).handleAction(
  CHANGE_FILTER,
  (state, action) => action.payload
);

export default combineReducers({
  todos,
  todosFilter,
});

```

[⇧ back to top](#table-of-contents)



[⇧ back to top](#table-of-contents)

---

## Selectors with `reselect`

```tsx
import { createSelector } from 'reselect';

import { TodosState } from './reducer';

export const getTodos = (state: TodosState) => state.todos;

export const getTodosFilter = (state: TodosState) => state.todosFilter;

export const getFilteredTodos = createSelector(getTodos, getTodosFilter, (todos, todosFilter) => {
  switch (todosFilter) {
    case 'completed':
      return todos.filter(t => t.completed);
    case 'active':
      return todos.filter(t => !t.completed);

    default:
      return todos;
  }
});

```

[⇧ back to top](#table-of-contents)

---

## Connect with `react-redux`

### Typing connected component

*__NOTE__: Below you'll find only a short explanation of concepts behind typing `connect`. For more real-world examples please check [Redux Connected Components](#redux-connected-components) section.*

```tsx
import MyTypes from 'MyTypes';

import { bindActionCreators, Dispatch, ActionCreatorsMapObject } from 'redux';
import { connect } from 'react-redux';

import { countersActions } from '../features/counters';
import { FCCounter } from '../components';

// Type annotation for "state" argument is mandatory to check 
// the correct shape of state object and injected props you can also
// extend connected component Props interface by annotating `ownProps` argument
const mapStateToProps = (state: MyTypes.RootState, ownProps: FCCounterProps) => ({
  count: state.counters.reduxCounter,
});

// "dispatch" argument needs an annotation to check the correct shape
//  of an action object when using dispatch function
const mapDispatchToProps = (dispatch: Dispatch<MyTypes.RootAction>) =>
  bindActionCreators({
    onIncrement: countersActions.increment,
  }, dispatch);

// shorter alternative is to use an object instead of mapDispatchToProps function
const dispatchToProps = {
    onIncrement: countersActions.increment,
};

// Notice we don't need to pass any generic type parameters to neither
// the connect function below nor map functions declared above
// because type inference will infer types from arguments annotations automatically
// This is much cleaner and idiomatic approach
export const FCCounterConnected =
  connect(mapStateToProps, mapDispatchToProps)(FCCounter);

// You can add extra layer of validation of your action creators
// by using bindActionCreators generic type parameter and RootAction type
const mapDispatchToProps = (dispatch: Dispatch<MyTypes.RootAction>) =>
  bindActionCreators<ActionCreatorsMapObject<Types.RootAction>>({
    invalidActionCreator: () => 1, // Error: Type 'number' is not assignable to type '{ type: "todos/ADD"; payload: Todo; } | { ... }
  }, dispatch);

```


# Configuration & Dev Tools

## TSLib
https://www.npmjs.com/package/tslib

This library will cut down on your bundle size, thanks to using external runtime helpers instead of adding them per each file.

> Installation  
`npm i tslib`

Then add this to your `tsconfig.json`:
```ts
"compilerOptions": {
  "importHelpers": true
}
```

[⇧ back to top](#table-of-contents)


## ESLint


## Jest


## Guidelines & Style Guide 
### ["Dave Mobile App Development Guidelines"](https://docs.google.com/document/d/1JRGrpDjxvkZtdjxhydBhITdpA1c8DTU2cm8b3O8LCOc/edit#heading=h.7s84esr9iihi)
### ["react-styleguidist"](https://github.com/styleguidist/react-styleguidist)

[⟩⟩⟩ styleguide.config.js](/playground/styleguide.config.js)  

[⟩⟩⟩ demo](https://piotrwitek.github.io/react-redux-typescript-guide/)

[⇧ back to top](#table-of-contents)

---

# Recipes

### General Tips

#### - should I still use React.PropTypes in TS?
No. With TypeScript, using PropTypes is an unnecessary overhead. When declaring Props and State interfaces, you will get complete intellisense and design-time safety with static type checking. This way you'll be safe from runtime errors and you will save a lot of time on debugging. Additional benefit is an elegant and standardized method of documenting your component public API in the source code.  

[⇧ back to top](#table-of-contents)

#### - when to use `interface` declarations and when `type` aliases?
From practical side, using `interface` declaration will create an identity (interface name) in compiler errors, on the contrary `type` aliases doesn't create an identity and will be unwinded to show all the properties and nested types it consists of.  
Although I prefer to use `type` most of the time there are some places this can become too noisy when reading compiler errors and that's why I like to leverage this distinction to hide some of not so important type details in errors using interfaces identity.
Related `ts-lint` rule: https://palantir.github.io/tslint/rules/interface-over-type-literal/  

[⇧ back to top](#table-of-contents)

#### - what's better default or named exports?
A common flexible solution is to use module folder pattern, because you can leverage both named and default import when you see fit.  
With this solution you'll achieve better encapsulation and be able to safely refactor internal naming and folders structure without breaking your consumer code:

```ts
// 1. create your component files (`select.tsx`) using default export in some folder:

// components/select.tsx
const Select: React.FC<Props> = (props) => {
...
export default Select;

// 2. in this folder create an `index.ts` file that will re-export components with named exports:

// components/index.ts
export { default as Select } from './select';
...

// 3. now you can import your components in both ways, with named export (better encapsulation) or using default export (internal access):

// containers/container.tsx
import { Select } from '@src/components';
or
import Select from '@src/components/select';
...
```





### Type-Definitions Tips



---

## Tutorials & Articles
> Curated list of relevant in-depth tutorials

Higher-Order Components:
- https://medium.com/@jrwebdev/react-higher-order-component-patterns-in-typescript-42278f7590fb

[⇧ back to top](#table-of-contents)
---
