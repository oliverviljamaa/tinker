# :rotating_light: AngularJS test
[![npm](https://img.shields.io/npm/v/angularjs-test.svg)](https://www.npmjs.com/package/angularjs-test) [![GitHub release](https://img.shields.io/github/release/oliverviljamaa/angularjs-test.svg)](https://github.com/oliverviljamaa/angularjs-test/releases) [![CircleCI](https://img.shields.io/circleci/project/github/oliverviljamaa/angularjs-test/master.svg)](https://circleci.com/gh/oliverviljamaa/angularjs-test) [![npm](https://img.shields.io/npm/l/angularjs-test.svg)](https://github.com/oliverviljamaa/angularjs-test/blob/master/LICENSE)

Unit testing utility for [AngularJS (1.x)](https://angularjs.org/), heavily inspired by the wonderful [Enzyme](http://airbnb.io/enzyme/) API. :heart:  
Therefore, it is well suited for organisations and individuals **moving from AngularJS to React**. It is **test framework and runner agnostic**, but the examples are written using [Jest](https://github.com/facebook/jest) syntax.

Available methods:  
[`mount`](#mounttemplate-props--testelementwrapper)  
[`mockComponent`](#mockcomponentname--mock)

Returned classes:  
[`TestElementWrapper`](#testelementwrapper-api)  
[`mock`](#mock-api)

## Usage

```bash
npm install angularjs-test --save-dev
```

### Module context

```js
import { mount, mockComponent } from 'angularjs-test'; 
```

### Non-module context

1. Include the script from `node_modules/angularjs-test/dist/angular-test.js`.
2. Use the utility from the global context under the name `angularjsTest`.

## API

### `mount(template, [props]) => TestElementWrapper`

Mounts the `template` (`String`) with optional `props` (`Object`) and returns a [`TestElementWrapper`](#testelementwrapper-api) with numerous helper methods. The props are attached to the `$ctrl` available in the scope.

<details>
  <summary>Example</summary>

```js
import 'angular';
import 'angular-mocks';
import { mount } from 'angularjs-test';

describe('Component under test', () => {
  const TEMPLATE = `
    <h1>{{ $ctrl.title }}</h1>
    <p>{{ $ctrl.text }}</p>
  `;

  let component;
  beforeEach(() => {
    angular.mock.module('moduleOfComponentUnderTest');
    component = mount(TEMPLATE, { title: 'A title', text: 'Some text' });
  });
});
```

</details>

### `mockComponent(name) => mock`

By default, AngularJS renders the whole component tree. This function mocks a child component with `name` (`String`) in the component under test and returns a [`mock`](#mock-api). The child component won't be compiled and its controller won't be invoked, enabling testing the component under test in isolation. In addition, the returned `mock` has methods useful for testing.

<details>
  <summary>Example</summary>

```js
import 'angular';
import 'angular-mocks';
import { mockComponent } from 'angularjs-test';

describe('Component under test', () => {
  let childComponent;
  beforeEach(() => {
    angular.mock.module('moduleOfComponentUnderTest');
    childComponent = mockComponent('child-component'); // ⇦ after module, before inject
  });
});
```

</details>

### `TestElementWrapper` API

#### `.length => Number`

The number of elements in the wrapper.

<details>
  <summary>Example</summary>

```js
let component;
beforeEach(() => {
  component = mount(`
    <ul>
      <li>1</li>
      <li>2</li>
      <li>3</li>
    </ul>
  `);
});

it('has three list items', () => {
  expect(component.find('li').length).toBe(3);
});
```

</details>

### `mock` API

#### `.exists() => Boolean`

Returns whether or not the mocked component exists in the rendered template.

<details>
  <summary>Example</summary>

```js
let component;
beforeEach(() => {
  component = mount(`
    <button ng-click="$ctrl.show = !$ctrl.show">
      Show child
    </button>
    <child-component ng-if="$ctrl.show"></child-component>
  `);
});

it('allows toggling child component', () => {
  const button = component.find('button');

  expect(childComponent.exists()).toBe(false);
  button.simulate('click');
  expect(childComponent.exists()).toBe(true);
  button.simulate('click');
  expect(childComponent.exists()).toBe(false);
});
```

</details>

#### `.props() => Object`

Returns all mocked component props.

<details>
  <summary>Example</summary>

```js
let component;
beforeEach(() => {
  component = mount(`
    <div>Something else</div>
    <child-component
      some-prop="'A string'",
      some-other-prop="12345"
    ></child-component>
  `);
});

it('passes props to child component', () => {
  expect(childComponent.props()).toEqual({
    someProp: 'A string',
    someOtherProp: 12345,
  });
});
```

</details>

#### `.prop(key) => Any`

Returns mocked component prop value with the provided `key`.

<details>
  <summary>Example</summary>

```js
let component;
beforeEach(() => {
  component = mount(`
    <div>Something else</div>
    <child-component some-prop="'A string'"></child-component>
  `);
});

it('passes some prop to child component', () => {
  expect(childComponent.prop('someProp')).toBe('A string');
});
```

</details>

#### `.simulate(event, [data]) => Any`

Calls an event handler on the mocked component for passed `event` with `data` (optional).

NOTE: `event` should be written in camelCase and without the `on` present in the event handler name. So, to call `onSomePropChange`, `.simulate('somePropChange')` should be used.

<details>
  <summary>Example</summary>

```js
it('calls parent component with data when child component is called', () => {
  const onSomePropChange = jest.fn();
  mount(
    `
      <div>Something else</div>
      <child-component
        on-some-prop-change="onSomePropChange($event)"
      ></child-component>
    `,
    { onSomePropChange } // ⇦ props for component under test
  );

  expect(onSomePropChange).not.toBeCalled();
  childComponent.simulate('somePropChange', 'New value');
  expect(onSomePropChange).toBeCalledWith('New value');
});
```

</details>

## Contributing

1. Run tests with `npm run test:watch`. `npm test` will check for package and changelog version match, ESLint and Prettier format in addition.
1. Bump version number in `package.json` according to [semver](http://semver.org/) and add an item that a release will be based on to `CHANGELOG.md`.
1. Submit your pull request from a feature branch and get code reviews.
1. If the pull request is approved and the [CircleCI build](https://circleci.com/gh/oliverviljamaa/angularjs-test) passes, you will be able to squash/rebase and merge.
1. Code will automatically be released to [GitHub](https://github.com/oliverviljamaa/angularjs-test/releases) and published to [npm](https://www.npmjs.com/package/angularjs-test) according to the version specified in the changelog and `package.json`.

## Other

For features and bugs, feel free to [add issues](https://github.com/oliverviljamaa/angularjs-test/issues) or contribute.
