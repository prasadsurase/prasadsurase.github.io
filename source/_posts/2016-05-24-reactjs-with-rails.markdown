---
layout: post
title: "Reactjs with Rails"
date: 2016-05-24 20:11:18 +0530
comments: true
categories: [reactjs, rails]
---

Since last couple of days, I have been experimenting with ReactJS. I followed the official tutorial, Thinking in React 
and some other blogs. This post is a summary of what I learned. I will try to keep is as simple as possible.

React follows **unidirectional data flow**. To understand this, you first need to know Component, state and 
props are.

**What is a Component?**
A component is an entity that is supposed to do only one thing. Ideally, it should follow Single Responsibility Principle. The main 
task of a component is to render data(props/state).

**What is state?**
A component's data that it has not inherited from the Parent Component is called state. 

**What is props?**
Properties(props) that a Component inherits from the Parent/Ancestor Component is called props. 

What it means is that the data(state/props) that a component passes down to child components as props is 


A simple 'Hello React' program. 

```javascript
var HelloWorld = React.createClass({
  getInitialState: function(){
    return {greeting: 'Hello'}
  },

  render: function(){
    return (
      <h3>{this.state.greeting} {this.props.name}</h3>
    );
  }
});

ReactDOM.render(
  <HelloWorld name="React" />,
  document.getElementById('container')
);
```

Here, the parent component is **ReactDOM** and the child component is **HelloWorld**. **greeting** is the **state** of HelloWorld 
whereas **name** is its **props**.

* A component can alter(update/delete) it's state since it belongs to the component. When state is passed down to child components, it becomes props for child components.
* Props are inherited and hence should not be altered.


* State of Parent component will be the props of child component.
* Props are passed to child components along with the callbacks that need to be called to update/delete the props.
* Child components should not update/delete the Parent components data(props)
* React will update all the children and their data once the Parent components data(props) has been updated.
* Props are immutable.
* States are mutable.
* Components current data is its state. When passed down to child components, it becomes props for those components
* Props is used when data doesnt change.
* State is used when data might change.
* Components convert data to html.
* Props + State => data
* Update a component's state using 'setStat()'. To update a parent components data, call the callback methods provided 
  by the parent component passing the updated data as arguments.
* If component's data is going to be altered in the future, it should be set as state.
* When child component receives data(props) from the parent, do not assign it to child component's state and then use 
  these states to render html. Use the props directly in the render function. The state is set when the component is 
  mounted for the first time. you will have to explicitly unmount and remount the child component to rerender the update 
  state data.
* Child can call Parent component's only those methods that have been passed on to it.
* If a state is refered in multiple components of different types, the state should be added to the most recent ancestor 
  of the components unless the components are not in the same inheritance chain.


All the code samples are supposed to be inside the 'script' tag. The basic HTML page to get started can be downloaded from [Getting Started](https://facebook.github.io/react/docs/getting-started.html)
