---
layout: post
title: "Basic concepts of React"
date: 2016-05-24 20:11:18 +0530
comments: true
categories: [reactjs, rails]
---

Since last couple of days, I have been experimenting with ReactJS. I followed the official 
[tutorial](https://facebook.github.io/react/docs/tutorial.html), [Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) 
and some other blogs. This post is a summary of what I learned. I will try to keep is as simple as possible.

**What is React?** ReactJS is a JavaScript library to manage and render Views(V in MVC). The views are rendered by React
Components.

Lets read some simple code. A simple 'Hello React' program. 

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title>Hello React!</title>
    <script src="build/react.js"></script>
    <script src="build/react-dom.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/babel">
      var HelloWorld = React.createClass({
        getInitialState: function(){
          return {message: 'Hello'}
        },

        render: function(){
          return (
            <h3>{this.state.message} {this.props.name}</h3>
          );
        }
      });

      ReactDOM.render(
        <HelloWorld name="React" />,
        document.getElementById('container')
      );
    </script>
  </body>
</html>
```
We have a HTML page with some JS code defined inside the _script_ tag. The JS code has some HTML too. This HTML inside JS is 
called [JSX](https://jsx.github.io). [Babel](https://babeljs.io/) is a library to parse JSX. React uses JSX to render the HTML.

Coming to the JS code, we have 2 entities named **ReactDOM** and **HelloWorld**. In React, these are called **Component**. In the 
above code, the **ReactDOM** is calling the **HelloWorld** component passing some data(ie. name="React")

Here, the parent component is **ReactDOM** and the child component is **HelloWorld**. **message** is the **state** of HelloWorld 
whereas **name** is its **props**.

**What is a Component?**
A component is an entity that is supposed to do only one thing. Ideally, it should follow _Single Responsibility Principle_. The 
main task of a component is to render data(props/state). Component don't inherit other Component but data(state/props) is passed 
form one Component(Parent) to another Component(Child).

**What is state?**
A Component's data that it has not inherited from the Parent Component is called state.

**What is props?**
Data that a Component inherits from the Parent/Ancestor Component is called props(properties). 

React follows **unidirectional data flow**. What it means is that a Component(Parent) passes data(state/props) to another 
Component(Child). When the data of the Parent is altered, the Child component automatically update itself.

**Some important points:**

* Component _must_ implement the **render** function. Component converts data to JSX.
* When **state** is passed down to child components, it becomes **props** for child components. ie. state of Parent Component will 
  always be props of Child Component.
* A component can alter(update/delete) it's state since it belongs to the component. **_state is mutable_**.
* A component should not alter(update/delete) it's props since they don't belong to the component. **_props is immutable_**.
* If props are to be altered, then they should be altered in the component where they are declared. When mounting the Child 
  Component, the callbacks that need to be called to update/delete the props should be passed too.
* React will update all the children using the props once the Parent component's data(props) has been updated.
* When child component receives data(props) from the parent, do not assign it to child component's state and then use this state
  to render html. Use the props directly in the render function. The state is set when the component is mounted for the first 
  time. you will have to explicitly unmount and remount the child component to rerender the update state data.
* Child Component can call Parent Component's only those methods that have been passed on to it.
* If a state is refered in multiple components of different types, the state should be added to the most recent ancestor of the 
  components.

Lets make sure that these points are right. We will create a simple example wherein a User can enter _message_(eg. 'Hello') and a 
_name_(eg. 'Prasad'). The page should display the _message_ and _name_.

There are couple of points that yyou should always remember. Rather than listing them all out at once, lets create an example 
step-by-step and see what we find out.

For sake of keeping this post short and simple, I won't be adding the entire HTML but only the necessary(updated) code. Also, the code 
sample would contain only the changes for the components.

```javascript
var Greet = React.createClass({});

ReactDOM.render(<Greet />, document.getElementById('container'));
```

The above code sample will fail because every Component created using _React.createClass_ must implement the _render_ function. 
The detailed error can be viewed in the browser's console.
__

```javascript
var Greet = React.createClass({
  getInitialState: function(){
    return { message: 'Hello' };
  },

  render: function(){
    return (
      <h3>{this.state.message}</h3>
    );
  }
});
```
We updated the **Greet** component to set some initial value for _message_ and defined the _render_ function to display the 
_message_. Since, _message_ has been defined in the same component, hence, _message_ is _Greet's_ state.

```javascript
var Greet = React.createClass({
  ...
  render: function(){
    return (
      <User message={this.state.message} />
    );
  }
});

var User = React.createClass({
  render: function(){
    return (
      <h3>{this.props.message}</h3>
    );
  }
});
```
Here, we created a new component named *User* and passed the Greet's message to the User. Since, _message_ is passed to User, 
it's User's props.

```javascript
var User = React.createClass({
  getInitialState: function(){
    return { name: 'Prasad' };
  },

  render: function(){
    return (
      <div>
        <input type='text' name='message' placeholder='Enter message' value={this.props.message}/>
        <input type='text' name='name' placeholder='Enter Name' value={this.state.name}/>
        <h3>{this.props.message} {this.state.name}</h3>
      </div>
    );
  }
});
```
Here, we added a state to User named _name_. We are rendering the props message and state name in the User's render function 
along with 2 input fields wherein User can enter the custom message and name that he/she wants to be displayed.

```javascript
var User = React.createClass({
  handleNameChange: function(event){
    this.setState({ name: event.target.value });
  },

  render: function(){
    return (
      <div>
        <input type='text' placeholder='Enter message' value={this.props.message} />
        <input type='text' placeholder='Enter Name' value={this.state.name} onChange={this.handleNameChange}/>
        <h3>{this.props.message} {this.state.name}</h3>
      </div>
    );
  }
});
```
Here, we added the functionality so that we can specify the name that we want to be displayed along with the message. The 
handleNameChange function receives the event and retrieves the updated value from the target. React provides us a function named
setState using which we have updated the state name. If we update the state name without using the setState function, the updated 
name is not displayed. This is because the setState informs React that an state has been updated and that the DOM needs to be 
updated.

```javascript
var Greet = React.createClass({
  updateMessage: function(value){
    this.setState({ message: value });
  },

  render: function(){
    return (
      <User message={this.state.message} handleMessageChange={this.updateMessage}/>
    );
  }
});

var User = React.createClass({
  handleGreetChange: function(e){
    this.props.handleMessageChange(e.target.value);
  },

  render: function(){
    return (
      <div>
        <input type='text' placeholder='Enter message' value={this.props.message} onChange={this.handleGreetChange}/>
        <input type='text' placeholder='Enter Name' value={this.state.name} onChange={this.handleNameChange}/>
        <h3>{this.props.message} {this.state.name}</h3>
      </div>
    );
  }
```
Here, we added a function called updateMessage to Greet. This function will receive the updated value and will update the 
state message. Also, we need to pass the function to User component. When the user types the message in the textbox, the 
handleGreetChange function is called. This function retrieves the entered value and passes it handleGreetChange props as a 
parameter.

So, the entire code is as

```javascript
var Greet = React.createClass({
  getInitialState: function(){
    return { message: 'Hola' };
  },

  updateMessage: function(value){
    this.setState({message: value});
  },

  render: function(){
    return (
    <User message={this.state.message} handleMessageChange={this.updateMessage}/>
    );
  }
});

var User = React.createClass({
  getInitialState: function(){
    return { name: 'Prasad' }
  },

  handleNameChange: function(e){
    this.setState({name: e.target.value});
  },

  handleGreetChange: function(e){
    this.props.handleMessageChange(e.target.value);
  },

  render: function(){
    return (
      <div>
        <input type='text' placeholder='Enter Message' value={this.props.message} onChange={this.handleGreetChange}/>
        <input type='text' placeholder='Enter Name' value={this.state.name} onChange={this.handleNameChange}/>
        <h3>
          {this.state.some} {this.props.message} {this.state.name}
        </h3>
      </div>
    );
  }
});

ReactDOM.render(
  <Greet/>,
  document.getElementById('container')
);
```
The User component defined above can be optimized a bit as below

```javascript
var User = React.createClass({
  getInitialState: function(){
    return { name: 'Prasad' }
  },

  handleNameChange: function(e){
    if(e.target.name == "message"){                                                                                         
      this.props.handleMessageChange(e.target.value);                                                                       
    } else {                                                                                                                
      this.setState({name: e.target.value});                                                                                
    }  
  },

  render: function(){
    return (
      <div>
        <input type='text' name="message" placeholder='Enter Message' value={this.props.message} onChange={this.handleGreetChange}/>
        <input type='text' name="name" placeholder='Enter Name' value={this.state.name} onChange={this.handleNameChange}/>
        <h3>
          {this.state.some} {this.props.message} {this.state.name}
        </h3>
      </div>
    );
  }
});
```
