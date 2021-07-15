# Seperating logic, state and layout

React is a powerful framework for integrating javascript logic and the HTML DOM.
This has significant advantages for writing data driven apps where DOM components are directly driven
by the current state.

The downside to this is the bluring of the boundary between logic, state and DOM layout.
Understanding the differences between these three app layers is important for creating
testable and maintainable code.

There are multiple inbuilt React functionality and plugins for organising logic, state and layout including
react hooks, Redux, react context etc.

The ultimate aim is to be able to test the logic impact on state without worrying about rendering DOM and
to test the state impact on the DOM without worrying about the logic.

# A messy example

React jsx allows you to embed logic directly in your layout. This can be great for simple components
where seeing the logic alongside the layout is useful.
When components expand they become complex and the layout can become messy.

Issues caused by embedding logic in jsx layouts

- Reduces readability of logic
- More difficult to test
- Messy git diff when updating either logic or layout.

# The basic approach

The most simple way to clean up the above example is to extract the logic out of the jsx render function.

# React custom hooks and logic/ layout seperation

React hooks allow us to extract the logic of a component out of the component entirely while still
being able to take advantage of reacts powerful state management.

# Using Redux to extract the logic and state away from individual components

This method is most useful when part or all of your state is shared with many components such as user state.
