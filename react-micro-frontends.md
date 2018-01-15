# React Micro-Frontends

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
