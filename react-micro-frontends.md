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