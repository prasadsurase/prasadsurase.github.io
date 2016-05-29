---
layout: post
title: "Basic concepts of React"
date: 2016-05-24 20:11:18 +0530
comments: true
categories: [reactjs, rails]
published: true
---

Since last couple of days, I have been experimenting with ReactJS. I followed the official 
[Tutorial](https://facebook.github.io/react/docs/tutorial.html), [Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) 
and some other blogs. This post is a summary of what I learned. I will try to keep is as simple as possible.

**What is React**? React is a JavaScript library to manage and render Views(V in MVC). The views are rendered by React
**Component**s.

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
In the above code sample, we have a HTML page with some JS code defined inside the _script_ tag. The JS code has some HTML too. This HTML inside JS is 
called [JSX](https://jsx.github.io). [Babel](https://babeljs.io/) is a library to parse JSX. React uses JSX to render the HTML.

## Some basic terminologies:

**Component**: A component is an entity that is supposed to do only one thing. Ideally, it should follow **_Single Responsibility 
Principle_**. The main task of a component is to **render data**(props/*state*). Component doesn't inherit other Component but 
data(*state*/props) is passed form one Component(parent) to another Component(child).

**state**: A Component's data that it has not inherited from the parent Component is called state.

**props**: Data that a Component inherits from the parent/Ancestor Component is called props(properties). 

React follows **_unidirectional data flow_**. What it means is that a Component(parent) passes data(state/props) to another 
Component(child). When the data of the parent is altered, the child component automatically update itself.


In the javascript of the above code sample, we have 2 entities named **ReactDOM** and **HelloWorld**. In React, these are called 
**Component**. In the above code, the **ReactDOM** is calling the **HelloWorld** component passing some data(ie. name="React"). 
Hence, the parent component is **ReactDOM** and the child component is **HelloWorld**. **message** is the **state** of 
**HelloWorld** whereas **name** is its **props**.

Some important points:
---

* Component _must_ implement the _render_ function.
* When *state* is passed down to child components, it becomes _props_ for child components. ie. *state* of parent Component will 
  always be _props_ of child Component.
* A component can alter(update/delete) it's *state* since it belongs to the component. _state is mutable_.
* A component should not alter(update/delete) it's _props_ since they don't belong to the component. _props is immutable_.
* If _props_ are to be altered, then they should be altered in the component where they are declared. When mounting the child 
  Component, the callbacks that need to be called to update/delete the _props_ should be passed too.
* React will update all the child components using the _props_ once the parent component's data(*state*) has been updated.
* When child component receives data(props) from the parent, do not assign it to child component's *state* and then use this state
  to render html. Use the _props_ directly in the render function. The *state* is set when the component is mounted for the first 
  time. you will have to explicitly unmount and remount the child component to rerender the update *state* data.
* child Component can call parent Component's only those methods that have been passed on to it.
* If a *state* is refered in multiple components of different types, the *state* should be added to the most recent ancestor of the 
  components.

We will create a simple example wherein a User can enter _message_(eg. 'Hello') and a _name_(eg. 'Prasad'). The page should 
display the _message_ and _name_. 

For sake of keeping this post short and simple, I won't be adding the entire HTML but only the necessary(updated) code. 
Also, 1) The code sample would contain only the changes for the components. 2) We have the code sample followed by the explanation.


```javascript
var Greet = React.createClass({});

ReactDOM.render(<Greet />, document.getElementById('container'));
```

The above code sample will fail because every Component created using _React.createClass_ must implement the _render_ function. 
The detailed error can be viewed in the browser's console.

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
We updated the Greet component to set some initial value for _message_ and defined the _render_ function to display the 
_message_. Since, _message_ has been defined in the same component, hence, _message_ is _Greet's_ *state*.

```javascript
var Greet = React.createClass({
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
Here, we created a new component named *User* and passed the Greet's *message* to the User. Since, _message_ is passed to User, 
it's User's _props_.

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
Here, we added a *state* named _name_ to User. We are rendering the _props_ _message_ and *state* _name_ in the User's _render_ function 
along with 2 input fields wherein User can enter the custom _message_ and _name_ that he/she wants to be displayed.

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
Here, The _handleNameChange_ function receives the event and retrieves the updated value from the target. React provides us a 
function named _setState_ using which we have updated the *state* _name_. If we update the *state* _name_ without using the _setState_ 
function, the updated name is not displayed. This is because the _setState_ informs React that an *state* has been updated and that 
the DOM needs to be updated.

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
Here, the function _updateMessage_ will receive the updated value and will update the *state* _message_. Also, we need to pass the 
function to User component. When the user types the _message_ in the textbox, the _handleGreetChange_ function is called. This 
function retrieves the entered value and passes it _handleGreetChange_ _props_ as a parameter. Since, React implements 
unidirectional data flow, any components using the _props_ _message_ are updated with the updated value.

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
          {this.props.message} {this.state.name}
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
          {this.props.message} {this.state.name}
        </h3>
      </div>
    );
  }
});
```

### Some experiments

Ok. Since we have a working example, lets try to make come changes and check if they work or not. If not, why?

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
```

We didn't pass the _handleMessageChange_ _props_ to the User component. Now, when we enter the _message_, the same is not being 
displayed on the page. ie. Every parent component props(state/props/function) that you want to access in the child has to be 
explicitly passed on to the child Component.

```javascript
var User = React.createClass({
  render: function(){
    return (
      <div>
        <input type='text' name="message" placeholder='Enter Message' value={this.props.message} onChange={this.handleGreetChange}/>
        <input type='text' name="name" placeholder='Enter Name' value={this.state.name} onChange={this.handleNameChange}/>
        <h3>
          {this.state.some} {this.props.message} {this.state.name} {this.props.some}
        </h3>
      </div>
    );
  }
});
```

Here, we are accessing data(state/props) that has not been defined in any of the components. The above code will still work as 
expected.

```javascript
var Greet = React.createClass({
  updateMessage: function(value){
    this.setState({message: value});
    console.log('updated message: '+ this.state.message);
  }
});

var User = React.createClass({
  getInitialState: function(){
    return { name: 'Prasad', message: this.props.message }
  },

  render: function(){
    return (
      <div>
        <input type='text' placeholder='Enter Message' value={this.props.message} onChange={this.handleGreetChange}/>
        <input type='text' placeholder='Enter Name' value={this.state.name} onChange={this.handleNameChange}/>
        <h3>
          {this.state.message} {this.state.name}
        </h3>
      </div>
    );
  }
});
```
Here, we assigned the _message_ _props_ to a _message_ *state* and displayed the _message_ *state*. When we enter the new _message_ in the input 
box, the Greet's _message_ *state* is being updated(can be viewed in browser console) but the same in not reflected on the page. This 
is because the _getInitialState_ is called only when the component is mounted. Its like an initializer/constructor.

Summerizing, the post is about the the key concepts and their relations to get started with React. Hope it was interesting.

Critics are welcome.

