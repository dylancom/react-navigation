# React Native Tab View

[![Build Status][build-badge]][build]
[![Version][version-badge]][package]
[![MIT License][license-badge]][license]

A cross-platform Tab View component for React Native.

- [Run the example app to see it in action](https://expo.io/@satya164/react-native-tab-view-demos).
- Checkout the [example/](https://github.com/react-native-community/react-native-tab-view/tree/master/example) folder for source code.

## Features

- Smooth animations and gestures
- Scrollable tabs
- Supports both top and bottom tab bars
- Follows Material Design spec
- Highly customizable
- Fully typed with [Flow](https://flow.org/)

## Demo

<a href="https://raw.githubusercontent.com/satya164/react-native-tab-view/master/demo/demo.mp4"><img src="https://raw.githubusercontent.com/satya164/react-native-tab-view/master/demo/demo.gif" width="360"></a>

## Installation

Open a Terminal in the project root and run:

```sh
yarn add react-native-tab-view
```

You also need to install and link [react-native-gesture-handler](https://github.com/kmagiera/react-native-gesture-handler) and [react-native-reanimated](https://github.com/kmagiera/react-native-reanimated). Follow the instructions on the linked repos to configure them. This step is unnecessary if you use Expo.

## Quick Start

```js
import * as React from 'react';
import { View, StyleSheet, Dimensions } from 'react-native';
import { TabView, SceneMap } from 'react-native-tab-view';

const FirstRoute = () => (
  <View style={[styles.scene, { backgroundColor: '#ff4081' }]} />
);
const SecondRoute = () => (
  <View style={[styles.scene, { backgroundColor: '#673ab7' }]} />
);

export default class TabViewExample extends React.Component {
  state = {
    index: 0,
    routes: [
      { key: 'first', title: 'First' },
      { key: 'second', title: 'Second' },
    ],
  };

  render() {
    return (
      <TabView
        navigationState={this.state}
        renderScene={SceneMap({
          first: FirstRoute,
          second: SecondRoute,
        })}
        onIndexChange={index => this.setState({ index })}
        initialLayout={{ width: Dimensions.get('window').width }}
      />
    );
  }
}

const styles = StyleSheet.create({
  scene: {
    flex: 1,
  },
});
```

[Try this example on Snack](https://snack.expo.io/@satya164/react-native-tab-view-quick-start)

## More examples on Snack

- [Custom Tab Bar](https://snack.expo.io/@satya164/react-native-tab-view-custom-tabbar)
- [Lazy Load](https://snack.expo.io/@satya164/react-native-tab-view-lazy-load)

## Integration with React Navigation

React Navigation integration can be achieved by the [react-navigation-tabs](https://github.com/react-navigation/react-navigation-tabs) package. Note that while it's easier to use, it is not as flexible as using the library directly.

## API reference

The package exports a `TabView` component which is the one you'd use to render the tab view, and a `TabBar` component which is the default tab bar implementation.

### `TabView`

Container component responsible for rendering and managing tabs. Follows material design styles by default.

#### Example

```js
<TabView
  navigationState={this.state}
  onIndexChange={index => this.setState({ index })}
  renderScene={SceneMap({
    first: FirstRoute,
    second: SecondRoute,
  })}
/>
```

#### Props

##### `navigationState` (`required`)

State for the tab view. The state should contain the following properties:

- `index`: a number representing the index of the active route in the `routes` array
- `routes`: an array containing a list of route objects used for rendering the tabs

Each route object should contain the following properties:

- `key`: a unique key to identify the route (required)
- `title`: title for the route to display in the tab bar
- `icon`: icon for the route to display in the tab bar
- `accessibilityLabel`: accessibility label for the tab button
- `testID`: test id for the tab button

Example:

```js
{
  index: 1,
  routes: [
    { key: 'music', title: 'Music' },
    { key: 'albums', title: 'Albums' },
    { key: 'recents', title: 'Recents' },
    { key: 'purchased', title: 'Purchased' },
  ]
}
```

`TabView` is a controlled component, which means the `index` needs to be updated via the `onIndexChange` callback.

##### `onIndexChange` (`required`)

Callback which is called on tab change, receives the index of the new tab as argument.
The navigation state needs to be updated when it's called, otherwise the change is dropped.

##### `renderScene` (`required`)

Callback which returns a react element to render as the page for the tab. Receives an object containing the route as the argument:

```js
renderScene = ({ route, jumpTo }) => {
  switch (route.key) {
    case 'music':
      return <MusicRoute jumpTo={jumpTo} />;
    case 'albums':
      return <AlbumsRoute jumpTo={jumpTo} />;
  }
}
```

You need to make sure that your individual routes implement a `shouldComponentUpdate` to improve the performance. To make it easier to specify the components, you can use the `SceneMap` helper.

`SceneMap` takes an object with the mapping of `route.key` to React components and returns a function to use with `renderScene` prop.

```js
import { SceneMap } from 'react-native-tab-view';

...

renderScene = SceneMap({
  music: MusicRoute,
  albums: AlbumsRoute,
});
```

Specifying the components this way is easier and takes care of implementing a `shouldComponentUpdate` method.

Each scene receives the following props:

- `route`: the current route rendered by the component
- `jumpTo`: method to jump to other tabs, takes a `route.key` as it's argument

The `jumpTo` method can be used to navigate to other tabs programmatically:

```js
this.props.jumpTo('albums')
```

All the scenes rendered with `SceneMap` are optimized using `React.PureComponent` and don't re-render when parent's props or states change. If you need more control over how your scenes update (e.g. - triggering a re-render even if the `navigationState` didn't change), use `renderScene` directly instead of using `SceneMap`.

_IMPORTANT:_ **Do not** pass inline functions to `SceneMap`, for example, don't do the following:

```js
SceneMap({
  first: () => <FirstRoute foo={this.props.foo} />,
  second: SecondRoute,
});
```

Always define your components elsewhere in the top level of the file. If you pass inline functions, it'll re-create the component every render, which will cause the entire route to unmount and remount every change. It's very bad for performance and will also cause any local state to be lost.

##### `renderTabBar`

Callback which returns a custom React Element to use as the tab bar:

```js
import { TabBar } from 'react-native-tab-view';

...

renderTabBar = props => <TabBar {...props} />;
```

If this is not specified, the default tab bar is rendered. You pass this props to customize the default tab bar, provide your own tab bar, or disable the tab bar completely.

```js
renderTabBar = () => null;
```

##### `tabBarPosition`

Position of the tab bar in the tab view. Possible values are `'top'` and `'bottom'`. Defaults to `'top'`.

##### `swipeEnabled`

Boolean indicating whether to enable swipe gestures. Swipe gestures are enabled by default. Passing `false` will disable swipe gestures, but the user can still switch tabs by pressing the tab bar.

##### `swipeDistanceThreshold`

Minimum swipe distance which triggers a tab switch. By default, this is automatically determined based on the screen width.

##### `swipeVelocityThreshold`

Minimum swipe velocity which triggers a tab switch. Defaults to `1200`.

##### `initialLayout`

Object containing the initial height and width of the screens. Passing this will improve the initial rendering performance. For most apps, this is a good default:

```js
{ width: Dimensions.get('window').width }}
```

##### `sceneContainerStyle`

Style to apply to the view wrapping each screen. You can pass this to override some default styles such as overflow clipping:

##### `style`

Style to apply to the tab view container.

### `TabBar`

Material design themed tab bar. To pass props to the tab bar, you'd need to use the `renderTabBar` prop of `TabView` to render the `TabBar` and pass additional props.

#### Example

```js
renderTabBar={props =>
  <TabBar
    {...props}
    indicatorStyle={{ backgroundColor: 'pink' }}
  />
}
```

#### Props

##### `getLabelText`

Function which takes the current scene and returns the label text for the tab. Uses `route.title` by default.

```js
getLabelText={({ route }) => route.title}
```

##### `getAccessible`

Function which takes the current scene returns a boolean to indicate whether to mark a tab as `accessible`. Defaults to `true`.

##### `getAccessibilityLabel`

Function which takes the current scene and returns a accessibility label for the tab button. Uses `route.accessibilityLabel` by default if specified, otherwise uses the route title.

```js
getAccessibilityLabel={({ route }) => route.accessibilityLabel}
```

##### `testID`

Function which takes the current scene and returns a test id for the tab button to locate this tab button in tests. Uses `route.testID` by default.

```js
getTestID={({ route }) => route.testID}
```

getAccessibilityLabel: (props: { route: T }) => string;
Get accessibility label for the tab button. This is read by the screen reader when the user taps the tab.
 Uses `route.accessibilityLabel` by default if specified, otherwise uses the route title.

getTestID: (props: { route: T }) => string | undefined;
Get the id to locate this tab button in tests, uses `route.testID` by default.

##### `renderIcon`

Function which takes the current scene and returns a custom React Element to be used as a icon.

##### `renderLabel`

Function which takes the current scene and returns a custom React Element to be used as a label.

##### `renderIndicator`

Function which takes the current scene and returns a custom React Element to be used as a tab indicator.

##### `renderBadge`

Function which takes the current scene and returns a custom React Element to be used as a badge.

##### `onTabPress`

Function to execute on tab press. It receives the scene for the pressed tab, useful for things like scroll to top.

##### `onTabLongPress`

Function to execute on tab long press, use for things like showing a menu with more options

##### `pressColor`

Color for material ripple (Android >= 5.0 only).

##### `pressOpacity`

Opacity for pressed tab (iOS and Android < 5.0 only).

##### `scrollEnabled`

Boolean indicating whether to enable scrollable tabs.

##### `bounces`

Boolean indicating whether the tab bar bounces when scrolling.

##### `tabStyle`

Style to apply to the individual tabs in the tab bar.

##### `indicatorStyle`

Style to apply to the active indicator.

##### `labelStyle`

Style to apply to the tab item label.

##### `style`

Style to apply to the tab bar container.

## Optimization Tips

### Avoid unnecessary re-renders

The `renderScene` function is called every time the index changes. If your `renderScene` function is expensive, it's good idea move each route to a separate component if they don't depend on the index, and apply `shouldComponentUpdate` in your route components to prevent unnecessary re-renders.

For example, instead of:

```js
renderScene = ({ route }) => {
  switch (route.key) {
    case 'home':
      return (
        <View style={styles.page}>
          <Avatar />
          <NewsFeed />
        </View>
      );
    default:
      return null;
  }
};
```

Do the following:

```js
renderScene = ({ route }) => {
  switch (route.key) {
    case 'home':
      return <HomeComponent />;
    default:
      return null;
  }
};
```

Where `<HomeComponent />` is a `PureComponent`:

```js
export default class HomeComponent extends React.PureComponent {
  render() {
    return (
      <View style={styles.page}>
        <Avatar />
        <NewsFeed />
      </View>
    )
  }
}
```

### Avoid one frame delay

We need to measure the width of the container and hence need to wait before rendering some elements on the screen. If you know the initial width upfront, you can pass it in and we won't need to wait for measuring it. Most of the time, it's just the window width.

For example, pass the following `initialLayout` to `TabView`:

```js
const initialLayout = {
  height: 0,
  width: Dimensions.get('window').width,
};
```

The tab view will still react to changes in the dimension and adjust accordingly to accommodate things like orientation change.

### Optimize large number of routes

If you've a large number of routes, especially images, it can slow the animation down a lot. You can instead render a limited number of routes.

For example, do the following to render only 2 routes on each side:

```js
renderScene = ({ route }) => {
  if (Math.abs(this.state.index - this.state.routes.indexOf(route)) > 2) {
    return <View />;
  }

  return <MySceneComponent route={route} />;
};
```

### Avoid rendering TabView inside ScrollView

Nesting the `TabView` inside a vertical `ScrollView` will disable the optimizations in the `FlatList` components rendered inside the `TabView`. So avoid doing it if possible.

## Contributing

While developing, you can run the [example app](/example/README.md) to test your changes.

Make sure your code passes Flow and ESLint. Run the following to verify:

```sh
yarn flow
yarn lint
```

To fix formatting errors, run the following:

```sh
yarn lint -- --fix
```

Remember to add tests for your change if possible.

<!-- badges -->

[build-badge]: https://img.shields.io/circleci/project/github/react-native-community/react-native-tab-view/master.svg?style=flat-square
[build]: https://circleci.com/gh/react-native-community/react-native-tab-view
[version-badge]: https://img.shields.io/npm/v/react-native-tab-view.svg?style=flat-square
[package]: https://www.npmjs.com/package/react-native-tab-view
[license-badge]: https://img.shields.io/npm/l/react-native-tab-view.svg?style=flat-square
[license]: https://opensource.org/licenses/MIT