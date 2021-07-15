The beauty of the React framework was to bring together javascript and the DOM. Unfortunately this comes with many anti-patterns that are easy to fall into.


# 1. Hidden Logic

React component props allow you to specify a function as an input. Thanks to the nature of javascript we can write these functions inline. This can however lead to the following antipattern:
``` js
const Component = () => {
    return (
        <SubComponent
            handleSelect={() => {
                if (something) {
                    doSomething();
                } else {
                    doSomethingElse();
                }
            }}
        />
    )
}
```
Including component logic inside your component props adds an additional location for debugging. Instead I prefer to keep all component logic at the top of the component keeping the render section clean.

``` js
const Component = () => {
    const handleSelect = () => {
        if (something) {
            doSomething();
        } else {
            doSomethingElse();
        }
    }
    return (
        <SubComponent
            handleSelect={handleSelect}
        />
    )
}
```

Even better would be to hoist it out of the component itself to enable simpler testing.

``` js

export const handleSelect = () => {
    if (something) {
        doSomething();
    } else {
        doSomethingElse();
    }
}

const Component = () => {

    return (
        <SubComponent
            handleSelect={handleSelect}
        />
    )
}
```