# <img alt="React Intersection" src="https://user-images.githubusercontent.com/7850794/34434126-db688af8-ec7b-11e7-9527-a7a2c37edc3b.png" width="500">

**React Intersection** provides a simple component-based interface to the [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API).

It provides two core components: `IntersectionElement` and `IntersectionRoot`.

- **`IntersectionElement`** adds its direct DOM child as an observer of the nearest parent `IntersectionRoot`, or the browser viewport if none found.
- **`IntersectionRoot`** **optionally** creates a new `IntersectionObserver` with either the browser viewport or its direct DOM child as the observed `root`.

React Intersection is:

- **Tiny:** Less than 1kb.
- **Unopinionated:** Provides the full [`IntersectionObserverEntry`](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry) to each `onChange` handler. A great starting point for more specific functionality.
- **Component-based:** No need to provide selectors or `Node`s to set `IntersectionObserver.root`.

## Contents

- [Install](#Install)
- [Usage](#Usage)
- [API](#API)
- [Browser support](#Browser_support)
- [Suggested improvements](#Suggested_improvements)
- [Example](#Example)
- [Social](#Social)

## Install

### npm

```bash
npm install react-intersection --save
```

## Usage

### Observe an element's visibility

By default, wrapping a component with `IntersectionElement` will subscribe the first child DOM element to a default viewport `IntersectionObserver`, with `threshold` set to `[0, 1]`.

This means `onChange` will fire when the element is first fully off/on screen, and partially on/off screen.

```javascript
import { IntersectionElement } from 'react-intersection';

class Item extends React.Component {
  state = {
    isIntersecting: false
  };

  setVisibility = ({ isIntersecting }) => this.setState({ isIntersecting });

  render() {
    return (
      <IntersectionElement onChange={this.setVisibility}>
        <li />
      </IntersectionElement>
    );
  }
}
```

### Define a new root

We can use parent elements as the `IntersectionObserver.root` instead of the viewport:

```javascript
import { IntersectionRoot } from 'react-intersection';

const ScrollableList = () => (
  <IntersectionRoot>
    <ul>
      {renderItems()}
    </ul>
  </IntersectionRoot>
);
```

In the above example, `ul` will be the `root` of the new `IntersectionObserver` that any children `IntersectionElement`s will subscribe to.

### Define a new browser viewport root

To create a new browser viewport root with non-default settings, we can pass `viewport` to `IntersectionRoot`:

```javascript
const IntersectViewportWithMargins = ({ children }) => (
  <IntersectionRoot viewport margin="20px 20px 20px 20px">
    {children}
  </IntersectionRoot>
);
```

## API

### `IntersectionElement`

Observe when an element intersects with its closest `IntersectionRoot`

#### Props

##### `onChange: (e: IntersectionObserverEntry) => any`

Fires after every breached `threshold` on the `IntersectionRoot` (or browser viewport if none set).

Is provided an [IntersectionObserverEntry](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserverEntry) object.

##### `once?: boolean = false`

Stops firing events after the first `true` intersection.

### `IntersectionRoot`

Define a new IntersectionObserver.

**If no `root` property is defined, it only accepts a single child.**

#### Props

##### `viewport?: boolean = false`

If `true`, sets the browser viewport

##### `margin?: string = '0px 0px 0px 0px'`

A space-delimited list of margins that effectively change the observed bounding box.

Follows the CSS pattern of top/right/bottom/left where `'10px 20px 30px 40px'` would give a right margin of `20px`.

If `IntersectionRoot` is **not** a viewport observer, these values can be defined as percentages.

##### `threshold?: number[] = [0, 1]`

An array of values between `0` and `1` that dictates at which ratios the IntersectionObserver should fire callbacks. Thresholds are fully explained in the [MDN Intersection Observer API article](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API#Thresholds).

## Browser support

React Intersection is dependent on the browser Intersection Observer API and JavaScript's `WeakMap`:

### Intersection Observer API

[Browser support](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API#Browser_compatibility) | [Polyfill](https://www.npmjs.com/package/intersection-observer)

### WeakMap

[Browser support](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap#Browser_compatibility) | [Polyfill](https://www.npmjs.com/package/weakmap-polyfill)

## Suggested improvements

### Make `IntersectionRoot` optional

Currently, `IntersectionRoot` **must** be added at least once.

It'd be great if developers could simply use `IntersectionElement` which, if not the child of any `IntersectionRoot`, acted as if it were the child of:

```javascript
<IntersectionRoot viewport />
```

### `IntersectionElementChild`

Currently, `IntersectionElement` can be used to set something like an `isIntersecting` or `intersectionRatio` state, which can then be passed down to children via props.

It might be neat, convenient and performant to wrap larger components with `IntersectionElement` which smaller (and more numerous) `IntersectionElementChild` components could then subscribe to via context.

`IntersectionElementChild`'s `onChange` function would fire only when its parent `IntersectionElement` crosses the defined `threshold`.

For example:

```javascript
class LazyImage extends React.Component {
  state = {
    isVisible: false
  };

  checkVisibility = ({ isIntersecting }) => isIntersecting && this.setState({ isVisible: true });

  render() {
    const { src } = this.props;
    const { isVisible } = this.state;

    return (
      <IntersectionElementChild onChange={this.checkVisibility}>
        <img src={isVisible ? src : ''}/>
      </IntersectionElement>
    );
  }
}

const Product = ({ image, name, link }) => (
  <li>
    <a href={link}>
      <LazyImage src={image} />
    </a>
  </li>
); 

const ProductShelf = ({ products }) => (
  <IntersectionElement once>
    <ul>
      {products.map((product) => <Product {...product} />)}
    </ul>
  </IntersectionElement>
);
```

## Social

Follow DriveTribe Engineering on: [Medium](https://medium.com/drivetribe-engineering) | [Twitter](https://twitter.com/drivetribetech)

## Example

### Lazy loaded image

```javascript
class LazyLoadImage extends Component {
  state = {
    isVisible: false
  };

  checkVisibility = ({ isIntersecting }) => isIntersecting && this.setState({ isVisible: true });

  render() {
    const { src } = this.props;
    const { isVisible } = this.state;

    return (
      <IntersectionElement once onChange={this.checkVisibility}>
        <img src={isVisible ? src : ''}/>
      </IntersectionElement>
    );
  }
}

const Site = () => (
  <IntersectionRoot viewport>
    <LazyLoadImage src="path/to/image.jpg">
  </IntersectionRoot>
);
```
