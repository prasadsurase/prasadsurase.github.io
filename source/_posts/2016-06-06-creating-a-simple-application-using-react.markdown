---
layout: post
title: "Creating a simple application using React"
date: 2016-06-06 00:09:38 +0530
comments: true
published: true
categories: react
---

In the [previous](http://prasadsurase.github.io/blog/2016/05/29/basic-concepts-in-react/) post, we looked at the basic concepts in 
React. In this post, I would like to explain and demonstrate identifying and creating components with a simple example app.

We would be creating a simple React app as below:
{% img center /images/accounting.png 600 400 'image' 'images' %}

Here, we have 3 main sections: the summary of all the transactions, the form to enter the transaction details and the table 
displaying the transactions. So, we can say that we have 3 components. If the component becomes complex, we can futher 
break it down to multiple components. For now, this is enough for us to get started. Since all the 3 components are on the same level,
we will hold them collectively held under a component(say _TransactionsDetails_)

So, the hierarchy of components is going to be as

* _TransactionsDetails_
  * _TransactionsSummary_
  * _TransactionForm_
  * _TransactionsTable_
    * _TransactionRow_

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

First, we instruct React to render the base component(_TransactionsDetails_) at '#container'.

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

Then, we add the child components: _TransactionsSummary_, _TransactionForm_ & _TransactionsTable_. We are displaying 
_TransactionsSummary_ & _TransactionForm_ adjacent to each other. Below them, _TransactionsTable_ is displayed.

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

Now that the components skeleton is in place, lets start making changes for dynamic data. Lets fix the _TransactionForm_ first. 
Since, the _TransactionForm_ is the only place in the application from where data is going to be inserted, lets add states *date*, 
*reason* and *amount* to the _TransactionForm_ component.

```javascript
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
  },

  handleCancel: function(e){
    e.preventDefault();
    this.setState(this.getInitialState());
  },

  render: function(){
    return (
      <form onSubmit={this.handleSubmit}>
        Date: <input type='text' name='date' value={this.state.date} onChange={this.handleChange}/><br />
        Reason: <input type='text' name='reason' value={this.state.reason} onChange={this.handleChange}/><br />
        Amount: <input type='integer' name='amount' value={this.state.amount} onChange={this.handleChange}/><br />
        <button type='submit'>Submit</button>
        <button type='button' onClick={this.handleCancel}>Cancel</button>
      </form>
    );
  }
});
```
Now that we have added the basic states to the _TransactionForm_ and have functions to update form states when user enters data, 
we need to add logic to accept the form submit functionality. Here, after accepting the form data, we need to save the record in 
the _TransactionsDetails_ state since _TransactionsSummary_ and _TransactionsTable_ are the components that are going to refer to 
this state to update themselves.

```javascript
var TransactionForm = React.createClass({
  handleSubmit: function(e){
    e.preventDefault();
    this.props.handleNewRecord(this.state);
    this.setState(this.getInitialState());
  }
});

var TransactionsDetails = React.createClass({
  getInitialState: function(){
    return { records: [] };
  },

  addRecord: function(record){
    var records = this.state.records;
    records.push(record);
    this.setState({ records: records });
  },

  render: function(){
    return (
      ...
      <TransactionForm handleNewRecord={this.addRecord}/>
      ...
    );
  }
});
```
Since the form is working and the entered transactions are being saved in _TransactionsDetails_'s state.records, we move forward by 
displaying the dynamically added records in the table. For this, we introduce a new component named _TransactionRow_.

```javascript
var TransactionRow = React.createClass({
  render: function(){
    return (
      <tr>
        <td>{this.props.record.date}</td>
        <td>{this.props.record.reason}</td>
        <td>{this.props.record.amount}</td>
      </tr>
    );
  }
});

var TransactionsTable = React.createClass({
  render: function(){
    var rows = [];
    this.props.records.map(function(record, index){
      rows.push(<TransactionRow key={index} record={record} />);
    }.bind(this));
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
          { rows }
        </tbody>
      </table>
    );
  }
});

var TransactionsDetails = React.createClass({
  render: function(){
    return (
      ...
      <TransactionsTable records={this.state.records}/>
      ...
    );
  }
});
```
Now that we have the form and the table working, the only section remaining is the _TransactionsSummary_. Here, we are neither going 
to pass the state of _TransactionsDetails_ but are passing the return values of functions.

```javascript
var TransactionsSummary = React.createClass({
  render: function(){
    return(
      <ul>
        <li>Transactions: {this.props.tCount}</li>
        <li>Debits: 500</li>
        <li>Credits: 600</li>
        <li>Balance: 100</li>
        <li>Total: 1100</li>
      </ul>
    );
  }
});


var TransactionsDetails = React.createClass({
  transactionsCount: function(){
    return this.state.records.length;
  },

  render: function(){
    return (
      ...
      <TransactionsSummary tCount={this.transactionsCount()}/>
      ...
    );
  }
});
```

Similarly, we implement the credits, debits, balance and total transactions sumation as 

```javascript
var TransactionsSummary = React.createClass({
  render: function(){
    return(
      <ul>
        <li>Transactions: {this.props.tCount}</li>
        <li>Debits: {this.props.debits}</li>
        <li>Credits: {this.props.credits}</li>
        <li>Balance: {this.props.balance}</li>
        <li>Total: {this.props.total}</li>
      </ul>
    );
  }
});

var TransactionsDetails = React.createClass({
  debits: function(){
    var sum = 0;
    var records = this.state.records.filter(function(record){ return record.amount < 0});
    if(records.length > 0){
      sum = records.reduce((function(a, b) {
        return a + parseFloat(b.amount);
      }), 0);
    }
    return -sum;
  },

  credits: function(){
    var sum = 0;
    var records = this.state.records.filter(function(record){ return record.amount >= 0});
    if(records.length > 0){
      sum = records.reduce((function(a, b) {
        return a + parseFloat(b.amount);
      }), 0);
    }
    return sum;
  },

  balance: function(){
    return this.credits() - this.debits();
  },

  totalTransactions: function(){
    return this.credits() + this.debits();
  },

  render: function(){
    return (
      ...
      <TransactionsSummary tCount={this.transactionCount()} debits={this.debits()} credits={this.credits()}
      balance={this.balance()} total={this.totalTransactions()}/>
      ...
    );
  }
});
```
Since we have a working example, lets integrate bootstaps for better look and feel. So, our updated code is as 

```javascript
var TransactionsSummary = React.createClass({
  render: function(){
    return(
      <ul className="list-group">
        <li className="list-group-item">
          <span className="badge">{this.props.tCount}</span>
          Count
        </li>
        <li className="list-group-item">
          <span className="badge">{this.props.debits}</span>
          Debits
        </li>
        <li className="list-group-item">
          <span className="badge">{this.props.credits}</span>
          Credits
        </li>
        <li className="list-group-item">
          <span className="badge">{this.props.balance}</span>
          Balance
        </li>
        <li className="list-group-item">
          <span className="badge">{this.props.total}</span>
          Total
        </li>
      </ul>
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

  handleCancel: function(e){
    e.preventDefault();
    this.setState(this.getInitialState());
  },

  render: function(){
    return (
      <form onSubmit={this.handleSubmit} className="form-horizontal">
        <div className="form-group">
          <label for="date" className="col-sm-2 control-label">Date: </label>
          <div className="col-sm-10">
            <input type='text' name='date' className="form-control" value={this.state.date} onChange={this.handleChange}/>
          </div>
        </div>
        <div className="form-group">
          <label for="reason" className="col-sm-2 control-label">Reason: </label>
          <div className="col-sm-10">
            <input type='text' name='reason' className="form-control" value={this.state.reason} onChange={this.handleChange}/>
          </div>
        </div>
        <div className="form-group">
          <label for="amount" className="col-sm-2 control-label">Amount: </label>
          <div className="col-sm-10">
            <input type='integer' name='amount' className="form-control" value={this.state.amount} onChange={this.handleChange}/>
          </div>
        </div>
        <div className="form-group">
          <div className="col-sm-offset-2 col-sm-10">
            <button className="btn btn-primary" type='submit'>Submit</button>
            <button className="btn" type='button' onClick={this.handleCancel}>Cancel</button>
          </div>
        </div>
      </form>
    );
  }
});

var TransactionRow = React.createClass({
  render: function(){
    return (
      <tr>
        <td>{this.props.record.date}</td>
        <td>{this.props.record.reason}</td>
        <td>{this.props.record.amount}</td>
      </tr>
    );
  }
});

var TransactionsTable = React.createClass({
  render: function(){
    var rows = [];
    this.props.records.map(function(record, index){
      rows.push(<TransactionRow key={index} record={record} />);
    }.bind(this));
    return (
      <table className="table table-striped table-hover table-bordered">
        <thead>
          <tr>
            <th> Date </th>
            <th> Reason </th>
            <th> Amount </th>
          </tr>
        </thead>
        <tbody>
          { rows }
        </tbody>
      </table>
    );
  }
});

var TransactionsDetails = React.createClass({
  getInitialState: function(){
    return { records: [] };
  },

  addRecord: function(record){
    var records = this.state.records;
    records.push(record);
    this.setState({ records: records });
  },

  transactionCount: function(){
    return this.state.records.length;
  },

  debits: function(){
    var sum = 0;
    var records = this.state.records.filter(function(record){ return record.amount < 0});
    if(records.length > 0){
      sum = records.reduce((function(a, b) {
        return a + parseFloat(b.amount);
      }), 0);
    }
    return -sum;
  },

  credits: function(){
    var sum = 0;
    var records = this.state.records.filter(function(record){ return record.amount >= 0});
    if(records.length > 0){
      sum = records.reduce((function(a, b) {
        return a + parseFloat(b.amount);
      }), 0);
    }
    return sum;
  },

  balance: function(){
    return this.credits() - this.debits();
  },

  totalTransactions: function(){
    return this.credits() + this.debits();
  },

  render: function(){
    return (
      <div className="container">
        <div className='row'>
          <div className="col-md-offset-2 col-md-4">
            <div className="well">
              <TransactionsSummary tCount={this.transactionCount()} debits={this.debits()} credits={this.credits()}
              balance={this.balance()} total={this.totalTransactions()}/>
            </div>
          </div>
          <div className="col-md-4">
            <div className="well">
              <TransactionForm handleNewRecord={this.addRecord}/>
            </div>
          </div>
        </div>
        <div className='row'>
          <div className="col-md-offset-2 col-md-8">
            <div className="well">
              <TransactionsTable records={this.state.records}/>
            </div>
          </div>
        </div>
      </div>
    );
  }
});
```
Now that we have a working example that looks good, lets try to add the edit and delete options to alter the transactions. For 
this, we would be displaying the 'Edit' & 'Delete' buttons for each _TransactionRow_. When clicked on 'Delete' it should remove 
the record from table. When clicked 'Edit' it should load the transaction in _TransactionForm_ for editing.

```javascript
var TransactionRow = React.createClass({
  handleDelete: function(e){
    e.preventDefault();
    this.props.handleDeleteRecord(this.props.record);
  },

  render: function(){
    return (
      <tr>
        <td>{this.props.record.date}</td>
        <td>{this.props.record.reason}</td>
        <td>{this.props.record.amount}</td>
        <td>
          <button className="btn btn-primary" >Edit</button>
          <button className="btn btn-danger" onClick={this.handleDelete}>Delete</button>
        </td>
      </tr>
    );
  }
});

var TransactionsTable = React.createClass({
  render: function(){
    var rows = [];
    this.props.records.map(function(record, index){
      rows.push(<TransactionRow key={index} record={record} handleDeleteRecord={this.props.handleDeleteRecord}/>);
    }.bind(this));
    return (
      ...
    );
  }
});

var TransactionsDetails = React.createClass({
  getInitialState: function(){
    return { records: [
      { date: '1-6-2016', reason: 'Salary', amount: 1000},
      { date: '2-6-2015', reason: 'EMI', amount: -400}
    ] };
  },

  deleteRecord: function(record){
    var records = this.state.records;
    var index  = records.indexOf(record);
    records.splice(index, 1);
    this.setState({ records: records });
  },

  render: function(){
    return (
      <TransactionsTable records={this.state.records} handleDeleteRecord={this.deleteRecord}/>
    );
  }
});
```

Adding the functionality to edit the transaction record is going to be tricky because we need to inform another sibling component 
(_TransactionForm_) that it needs to load data for a selected record. We need to identify whether a record being editted in the 
form is new or existing. Hence, we add a new state *editing* to _TransactionForm_ which will be true if a record is being edited. 
Also, we add a state *recordToEdit* to _TransactionsDetails_. It points to record to be editted when 'Edit' is clicked.

```javascript
var TransactionForm = React.createClass({
  getInitialState: function(){
    return { record: { date: '', reason: '', amount: '' }, editing: false };
  },

  componentWillReceiveProps: function(nextProps) {
    this.setState({ record: nextProps.recordToEdit, editing: true });
  },
});

var TransactionRow = React.createClass({
  handleEdit: function(e){
    e.preventDefault();
    this.props.handleEditRecord(this.props.record);
  },

  render: function(){
    return (
      ...
      <button className="btn btn-primary" onClick={this.handleEdit} >Edit</button>
      ...
    )
  };
});

var TransactionsTable = React.createClass({
  render: function(){
    var rows = [];
    this.props.records.map(function(record, index){
      rows.push(<TransactionRow key={index} record={record} handleDeleteRecord={this.props.handleDeleteRecord} 
      handleEditRecord={this.props.handleEditRecord}/>);
    }.bind(this));
    return (
    ...
    );
  }
});

var TransactionsDetails = React.createClass({
  getInitialState: function(){
    return {
      records: [ { date: '1-6-2016', reason: 'Salary', amount: 1000}, { date: '2-6-2015', reason: 'EMI', amount: -400} ],
      recordToEdit: { date: '', reason: '', amount: '' }
    };
  },

  editRecord: function(record){
    this.setState({ recordToEdit: record });
  },

  render: function(){
    return (
      ...
      <TransactionForm handleNewRecord={this.addRecord} recordToEdit={this.state.recordToEdit}/>
      <TransactionsTable records={this.state.records} handleDeleteRecord={this.deleteRecord}
        handleEditRecord={this.editRecord}/>
      ...
    );
  }
});
```
The function 
*[componentWillReceiveProps](https://facebook.github.io/react/docs/component-specs.html#updating-componentwillreceiveprops)* is a 
lifecycle function provided by React. It is called when the component is being rerendered. It has the updated props that are 
passed as arguments to the function and can be accessed before the component is rerendered. 

Now, the last functionality that we need to add is updating a TransactionsTable. We have already implemented to load a transaction 
in the form so that it can be updated.

```javascript
var TransactionForm = React.createClass({
  handleChange: function(e){
    var record = this.state.record;
    switch(e.target.name){
      case 'date':
        record.date = e.target.value;
        break;
      case 'reason':
        record.reason = e.target.value;
        break;
      case 'amount':
        record.amount = e.target.value;
        break;
    }
    this.setState({ record: record });
  },

  handleSubmit: function(e){
    e.preventDefault();
    if(this.state.editing){
      this.props.handleUpdateRecord(this.state.record);
    } else {
      this.props.handleNewRecord(this.state.record);
    }
    this.setState(this.getInitialState());
  },

  render: function(){
    return (
      <form onSubmit={this.handleSubmit} className="form-horizontal">
        <h4>{this.state.editing ? 'Edit Record' : 'New Record'}</h4>
        ...
    );
  }
});

var TransactionsDetails = React.createClass({
  updateRecord: function(record){
    var records = this.state.records;
    var index = records.indexOf(record);
    records[index] = record;
    this.setState({ records: records, recordToEdit: {} });
  },

  render: function(){
    return (
      <TransactionForm handleNewRecord={this.addRecord} recordToEdit={this.state.recordToEdit}
        handleUpdateRecord={this.updateRecord}/>
    );
  }
});
```

Finally, we have a simple working application created using React. I know its not perfect and am definitely sure that there are 
some issues with it. But, our aim was to identitying, creating and linking the components. I hope the post achieved the aim. The 
source code can be found [here](https://github.com/prasadsurase/react-examples/blob/master/accounting.html).

Hope it was interesting. Critics are welcome.
