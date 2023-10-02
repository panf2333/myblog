# Note

## one-way data flow
React uses one-way data flow
> passing data down the component hierarchy from parent to child component


## props and state
here are two types of “model” data in React: props and state. The two are very different:

1. Props are like arguments you pass to a function. They let a parent component pass data to a child component and customize its appearance. For example, a Form can pass a color prop to a Button.
   
2. State is like a component’s memory. It lets a component keep track of some information and change it in response to interactions. For example, a Button might keep track of isHovered state.


## what is not state
1. Does it remain unchanged over time? 
   > If so, it isn’t state.
2. Is it passed in from a parent via props? 
   > If so, it isn’t state.
3. Can you compute it based on existing state or props in your component? 
   > If so, it definitely isn’t state!


## state live
1. Identify every component that renders something based on that state.
2. Find their closest common parent component—a component above them all in the hierarchy.
3. Decide where the state should live
   1. Often, you can put the state directly into their common parent.
   2. You can also put the state into some component above their common parent.
   3. If you can’t find a component where it makes sense to own the state, create a new component solely for holding the state and add it somewhere in the hierarchy above the common parent component.