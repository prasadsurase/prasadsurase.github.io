---
layout: post
title: "Identifying and creating Components in React"
date: 2016-05-31 19:21:06 +0530
comments: true
categories: react
---
In the [previous](http://prasadsurase.github.io/blog/2016/05/29/basic-concepts-in-react/) post, we looked at the basic concepts in 
React. In this post, I would like to explain and demonstrate identifying and creating components.

We would be creating a simple React app as below:

Here, we have 3 main sections: the summary of all the transactions, the form to enter the transaction details and the table 
displaying the transactions. So, we can say that we have 3 components. If the component becomes complex, we can futher 
break it down to multiple components. For now, this is enough for us to get started. All the 3 components can be collectively held 
under a component(say TransactionsDetails)

So, the hierarchy of components is going to be as

* TransactionsDetails
  * TransactionsSummary
  * TransactionForm
  * TransactionsTable
    * TransactionRow

For sake of simplicity, we will integrate Bootstrap after the entire app is up and running.

The basic HTML page is as below.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset='UTF-8' />
    <title>React Rocks!!!</title>
    <script src='build/react.js'></script>
    <script src='build/react-dom.js'></script>
    <script src='https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js'></script>
    <script src='https://ajax.googleapis.com/ajax/libs/jquery/2.2.2/jquery.min.js'></script>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/css/bootstrap.min.css">
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.6/js/bootstrap.min.js"></script>
  </head>
  <body>
    <div id='container'></div>
    <script type='text/babel'>
    </script>
  </body>
</html>
```

First, we instruct React to render the base component(TransactionsDetails) at '#container'.

```javascript
var TransactionsDetails = React.createClass({
  render: function(){
    return(
      null
    );
  }
});

ReactDOM.render( <TransactionsDetails />, document.getElementById('container'));
```

Then, we add the child components: TransactionsSummary, TransactionForm & TransactionsTable. We are displaying 
TransactionsSummary & TransactionForm adjacent to each other. Below them, TransactionsTable is displayed.

```javascript
var TransactionsSummary = React.createClass({
  render: function(){
    return (null);
  }
});

var TransactionForm = React.createClass({
  render: function(){
    return (null);
  }
});

var TransactionsTable = React.createClass({
  render: function(){
    return (null);
  }
});

var TransactionsDetails = React.createClass({
  render: function(){
    return (
      <div className="container">
        <div className='row'>
          <div className="col-md-3">
            <TransactionsSummary />
          </div>
          <div className="col-md-3">
            <TransactionForm />
          </div>
        </div>
        <div className='row'>
          <div className="col-md-6">
            <TransactionsTable />
          </div>
        </div>
      </div>
    );
  }
});
```

Now, lets start adding details for each component. I prefer displaying static data. Later, when all the components are in 
place and the static data is displayed properly, I start making changes for dynamic data.

```javascript
var TransactionsSummary = React.createClass({
  render: function(){
    return(
      <ul>
        <li>Transactions: 2</li>
        <li>Debits: 500</li>
        <li>Credits: 600</li>
        <li>Balance: 100</li>
        <li>Total: 1100</li>
      </ul>
    );
  }
});

var TransactionForm = React.createClass({
  render: function(){
    return (
      <form>
        Date: <input type='text' name='date' /><br />
        Reason: <input type='text' name='reason' /><br />
        Amount: <input type='integer' name='amount' /><br />
        <button type='submit'>Submit</button>
        <button type='button'>Cancel</button>
      </form>
    );
  }
});

var TransactionsTable = React.createClass({
  render: function(){
    return (
      <table>
        <thead>
          <tr>
            <th> Date </th>
            <th> Reason </th>
            <th> Amount </th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td> 31-05-2016 </td>
            <td> Salary </td>
            <td> 600 </td>
          </tr>
          <tr>
            <td> 01-06-2016 </td>
            <td> Party </td>
            <td> -500 </td>
          </tr>
        </tbody>
      </table>
    );
  }
});
```
Now that the skeleton is in place, lets add some data. Lets fix the form first. Since, the form is the only place in the 
application from where data is going to be inserted, lets add states date, reason and amount to the TransactionForm component.

```javascript
var TransactionsDetails = React.createClass({
  getInitialState: function(){
    return { records: [] }
  },

  addRecord: function(data){
    var records;                                                                                                            
    records = this.state.records.slice();                                                                                          
    records.push(record);                                                                                                   
    this.setState({ records: records }); 
  },

  render: function(){
    return (
      <div className="container">
        <div className='row'>
          <div className="col-md-3">
            <TransactionsSummary />
          </div>
          <div className="col-md-3">
            <TransactionForm handleNewRecord={this.addRecord}/>
          </div>
        </div>
        <div className='row'>
          <div className="col-md-6">
            <TransactionsTable/>
          </div>
        </div>
      </div>
    );
  }
});

var TransactionForm = React.createClass({
  getInitialState: function(){
    return { date: '', reason: '', amount: '' };
  },

  handleChange: function(e){
    switch(e.target.name){                                                                                                  
      case 'date':                                                                                                          
        this.setState({date: e.target.value});                                                                              
        break;                                                                                                              
      case 'reason':                                                                                                        
        this.setState({reason: e.target.value});                                                                            
        break;                                                                                                              
      case 'amount':                                                                                                        
        this.setState({amount: e.target.value});                                                                            
        break;                                                                                                              
    }   
  },

  handleSubmit: function(e){
    e.preventDefault();
    this.props.handleNewRecord(this.state);
    this.setState(this.getInitialState());
  },

  render: function(){
    return (
      <form onSubmit={this.handleSubmit}>
        Date: <input type='text' name='date' value={this.state.date} onChange={this.handleChange}/><br />
        Reason: <input type='text' name='reason' value={this.state.reason} onChange={this.handleChange}/><br />
        Amount: <input type='integer' name='amount' value={this.state.amount} onChange={this.handleChange}/><br />
        <button type='submit'>Submit</button>
        <button type='button'>Cancel</button>
      </form>
    );
  }
});
```
