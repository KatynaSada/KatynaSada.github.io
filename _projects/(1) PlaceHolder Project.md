---
name: Katyna Sada
tools: [C#, XML, WPF]
image: https://www.sketchappsources.com/resources/source-image/movie-badges-jurajjurik.png
description: This project has an individual showcase page, not just a direct link to the project site or repo. Now you have more space to describe your awesome project!
---

# The Movies Project

The Movies Project is something like **Netflix**, the only difference is that **it's not real**! It doesn't exist! I just created it to demonstrate how the **showcase** page looks like and how you can write whatever you want with full markdown support.

![preview](https://www.sketchappsources.com/resources/source-image/we-were-soldiers-landing-page-dbruggisser.jpg)

## Search Movies

![search](https://www.sketchappsources.com/resources/source-image/microsoft-windows-10-virtual-keyboard-diogo-sousa.png)

<p class="text-center">
{% include elements/button.html link="https://github.com/YoussefRaafatNasry/portfolYOU" text="Learn More" %}
</p>

## Components

Everything in React is a component, and these usually take the form of JavaScript classes. You create a component by extending upon the `React-Component` class. Let’s create a component called `Hello`.

```javascript
class Hello extends React.Component {
    render() {
        return <h1>Hello world!</h1>;
    }
}
```


You then define the methods for the component. In our example, we only have one method, and it’s called `render()`.

Inside `render()` you’ll return a description of what you want React to draw on the page. In the case above, we simply want it to display an `h1` tag with the text _Hello world!_ inside it.

To get our tiny application to render on the screen we also have to use `ReactDOM.render()`:

```javascript
ReactDOM.render(
    <Hello />,
    document.getElementById("root")
);
```
