# React Micro-Frontends

## Motivation
Reduce complexity, ease development and integration. Encourage the use of different technologies with a minimal coupling to the other parts of the application.

## Technical overview

### How do custom elements actually work?
The main idea is to use the [CustomElements API](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Custom_Elements) to achieve an optimal isolation of a component, providing the freedom to use whatever JS stack to implement a certain chunk of the whole application. Abstracted way of use:

```
<html>
    ...
    <script src="my-web-components/foo-component.js" />
    ...

    <div class="component-wrapper">
        <foo-component></foo-component>
    </div>
</html>
```

CustomElementsRegistry is included in the [custom-elements polyfill](https://github.com/webcomponents/custom-elements) and should be loaded globally on the page for this to work on firefox, without any further configuration. On chrome this already works out of the box.

```
<html>
    <head>
        <script src="custom-elements-polyfill.js">
        <script src="cmtf-application-bundle.js">
        <!--  Lazy loading for the custom elements themselves -->
    </head>
    ...
</html>
```

A custom element can receive props via attributes, similar to a React compoenent. Additionally it can expose API functions:

```
class MyComponent extends React.Component {

    handleMakeFooLaugh(){
        this.fooNode && this.fooNode.tickle()
    }

    render(){
        <div>
            <foo-component ref={node => this.fooNode = node}></foo-component>
            <button onClick={this.handleMakeFooLaugh}>Make foo laugh</button>
        </div>
    }

}
```

### Loading of a script of a custom element

The element's script should be loaded lazily from the wrapper element only if it wasn't loaded yet. For this purpose we need to create a general purpose wrapper:

```
type Props = {
    src: string, // Script URL to load the custom element
}

class CustomElement extends React.Component<Props, State> {

    onComponentDidMount() {
        // check if this custom-element was already loaded
        // if not load the script for it
    }

    render() {
        <>{
            this.props.children
        }</> // mount the custom-element
    }
}
```

Usage:

```
<CustomElement src="/customelements/foo-component.js">
    <foo-component 
        bar={this.props.barToPass}
        onclick={this.handleFooOnClick}>
    </foo-component>
</CustomElement>
```



# Caveat 

CustomElement stops working when babel transpiles to ES5. You need to use: https://github.com/github/babel-plugin-transform-custom-element-classes


## Integrating WebComponents into an existing React application

The challenge here is to gradually integrate the new WebComponents into the application without moving too many things
and without introducing a dependency hell. So even though ideally all WebComponents are in their own repository embedded
from their own node module, in this case we'll keep the all in the same repository as the mother application:

```
main-application |
                 | - application (contains the main React application
                 | - web-components |
                                    | - config.js
                                    | - foo-chat-application -|
                                                              | - package.json
                                    | - foo-shopping-cart -|
                                                           | - package.json
```

The next thing is taking care of the http servers that will actually deliver the custom-element scripts. We want to have
these servers running in 2 optional modes:

1. Combined development mode, when we're embedding the custom-element inside the mother application and want to be able
to make changes to both the mother application and the component. We're interested in having a file watcher to rebuild
and reload our changes instantly in both mother application and in the web-component. 
2. Serve only mode, when we're not making any changes to the custom-elements and just wish to have them served so that
we can work on the application or on another custom-elements. This mode has no file watchers in place and so uses less
system resources.

We should be able to run all web-components together, in all possible combinations so that we can work on one-or-more
components and have our changes reflected immediatly while saving resources and having other components served statically.

_Additionally_ we would like to be able to work on each component in complete isolation. This should be kept privately
by the component as an implementation detail. Typically, however we would like to see something like the following:

```
foo-chat-application $ yarn start
yarn start v0.22.0
$ webpack-dev-server 
Project is running at http://localhost:8080/
```

### Running a component 