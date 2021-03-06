[![Build Status](https://travis-ci.org/styled-components/jest-styled-components.svg?branch=master)](https://travis-ci.org/styled-components/jest-styled-components)
[![NPM version](https://img.shields.io/npm/v/jest-styled-components.svg)](https://www.npmjs.com/package/jest-styled-components)
[![tested with jest](https://img.shields.io/badge/tested_with-jest-99424f.svg)](https://github.com/facebook/jest)
[![styled with prettier](https://img.shields.io/badge/styled_with-prettier-ff69b4.svg)](https://github.com/prettier/prettier)

# Jest Styled Components
A set of utilities for testing [Styled Components](https://github.com/styled-components/styled-components) with [Jest](https://github.com/facebook/jest).
This package improves the snapshot testing experience and provides a brand new matcher to make expectations on the style rules.

# Quick Start

## Installation

```sh
yarn add --dev jest-styled-components
```

## Usage

```js
import React from 'react'
import styled from 'styled-components'
import renderer from 'react-test-renderer'
import 'jest-styled-components'

const Button = styled.button`
  color: red;
`

test('it works', () => {
  const tree = renderer.create(<Button />).toJSON()
  expect(tree).toMatchSnapshot()
  expect(tree).toHaveStyleRule('color', 'red')
})
```

# Snapshot Testing

Jest [snapshot testing](https://facebook.github.io/jest/docs/snapshot-testing.html) is an excellent way to test [React](https://facebook.github.io/react/) components (or any serializable value) and make sure things don't change unexpectedly.
It works with Styled Components but there are a few problems that this package addresses and solves.

For example, suppose we create this styled Button:

```js
import styled from 'styled-components'

const Button = styled.button`
  color: red;
`
```

Which we cover with the following test:

```js
import React from 'react'
import renderer from 'react-test-renderer'

test('it works', () => {
  const tree = renderer.create(<Button />).toJSON()
  expect(tree).toMatchSnapshot()
})
```

When we run our test command, Jest generates a snapshot containing a few class names (which we didn't set) and no information about the style rules:

```js
exports[`it works 1`] = `
<button
  className="sc-bdVaJa rOCEJ"
/>
`;
```

Consequently, changing the color to green:

```js
const Button = styled.button`
  color: green;
`
```

Results in the following diff, where Jest can only tell us that the class names are changed.
Although we can assume that if the class names are changed the style rules are also changed, this is not optimal ([and is not always true](https://github.com/styled-components/jest-styled-components/issues/29)).

```diff
- Snapshot
+ Received

 <button
-  className="sc-bdVaJa rOCEJ"
+  className="sc-bdVaJa hUzqNt"
 />
```

Here's where Jest Styled Components comes to rescue.

We just import the package into our test file:

```js
import 'jest-styled-components'
```

When we rerun the test, the output is different: the style rules are included in the snapshot, and the hashed class names are substituted with placeholders that make the diffs less noisy:

```diff
- Snapshot
+ Received

+.c0 {
+  color: green;
+}
+
 <button
-  className="sc-bdVaJa rOCEJ"
+  className="c0"
 />
```

This is the resulting snapshot:

```js
exports[`it works 1`] = `
.c0 {
  color: green;
}

<button
  className="c0"
/>
`;
```

Now, suppose we change the color again to blue:

```js
const Button = styled.button`
  color: blue;
`
```

Thanks to Jest Styled Components, Jest is now able to provide the exact information and make our testing experience even more delightful 💖:

```diff
- Snapshot
+ Received

 .c0 {
-  color: green;
+  color: blue;
 }

 <button
   className="c0"
 />
```

## Enzyme

[enzyme-to-json](https://www.npmjs.com/package/enzyme-to-json) is necessary to generate snapshots using [Enzyme](https://github.com/airbnb/enzyme)'s [shallow](http://airbnb.io/enzyme/docs/api/shallow.html) or [full DOM](http://airbnb.io/enzyme/docs/api/mount.html) rendering.

```sh
yarn add --dev enzyme-to-json
```

It can be enabled globally in the `package.json`:

```js
"jest": {
  "snapshotSerializers": [
    "enzyme-to-json/serializer"
  ]
}
```

Or imported in each test:

```js
import toJson from 'enzyme-to-json'

// ...

expect(toJson(wrapper)).toMatchSnapshot()
```

Jest Styled Components works with shallow rendering:

```js
import { shallow } from 'enzyme'

test('it works', () => {
  const wrapper = shallow(<Button />)
  expect(wrapper).toMatchSnapshot()
})
```

And full DOM rendering as well:

```js
import { mount } from 'enzyme'

test('it works', () => {
  const wrapper = mount(<Button />)
  expect(wrapper).toMatchSnapshot()
})
```

# toHaveStyleRule

The `toHaveStyleRule` matcher is useful to test if a given rule is applied to a component.
The first argument is the expected property, the second is the expected value.

```js
test('it works', () => {
  const tree = renderer.create(<Button />).toJSON()
  expect(tree).toHaveStyleRule('color', 'red')
})
```

This matcher works with trees serialized with `react-test-renderer` and shallow renderered or mounted with Enzyme.
It checks the style rules applied to the root component it receives, therefore to make assertions on components further in the tree they must be provided separately (Enzyme's [find](http://airbnb.io/enzyme/docs/api/ShallowWrapper/find.html) might help).

To use the `toHaveStyleRule` matcher with [React Native](https://facebook.github.io/react-native/), change the import statement to:

```js
import 'jest-styled-components/native'
```

# styled-components < v2

To use this package with styled-components < v2 (e.g. v1.4.6) the following annotation must be added at the top of the test files.
Consequently, it won't be possibile to use Enzyme's full DOM rendering.

```js
/**
 * @jest-environment node
 */
```

# Contributing

Please [open an issue](https://github.com/styled-components/jest-styled-components/issues/new) and discuss with us before submitting a PR.
