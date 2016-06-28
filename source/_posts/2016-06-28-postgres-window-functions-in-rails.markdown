---
layout: post
title: "Postgres Window Functions in Rails"
date: 2016-06-28 23:03:26 +0530
categories: [rails, postgres]
comments: true
published: true
---

I was working on a simple(I thought it was simple) feature wherein I wanted to list parent models. In the same table, particular 
child models for each parent were to be listed too. Say, a list of accounts, each with its latest 3 transactions. This feature 
turned out to be lot more tougher than I thought and was an excellent learning curve. Below are the details.

Lets say that we have a simple application consisting for 2 models, Account and Transaction.

```ruby
class Account
  has_many :transactions

  validates :account_number, presence: true
end

class Transaction
  belongs_to :account

  validates :account, presence: true
  validates :amount, presence: true
end
```
Now, as stated earlier, we need to list the accounts and last 3 transactions for each account.

```ruby
def index
  # for sake of simplicity, lets load first 5 accounts.
  @accounts = Account.limit(5)
end
```
```haml
# displaying the accounts
- @accounts.each do |account|
  = display the account's details
  - account.transactions.order(created_at :desc).limit(3).each do |t|
    = each transaction's details
```

This is not the right solution because it fires a query to load last 3 transactions for each account causing the N+1 problem. 
'including' the transactions in the _index_ was of no use too since that would load all the transactions for the 5 accounts and we 
just wanted the latest 3.

I had read somewhere that we can define relations as scopes. Since we can order/filter the data in the scope, I created a scope to 
load the required transactions.

```ruby
class Transaction
  belongs_to :account

  validates :account, presence: true
  validates :amount, presence: true

  scope :latest, -> { order(created_at: :desc).limit(3) }
end
```

Then, going ahead, decided to create a new association that would use to scope and would retrive only the required transactions 
for the account.

```ruby
class Account
  has_many :transactions
  has_many :latest_transactions, -> { latest }, class_name: Transaction

  validates :account_number, presence: true
end
```

Now, since we can _include_ an association for eager loading, I can definitely(I though so) use the newest association to load the 
accounts(first 5) and their required(latest 3) transactions.

```ruby
def index
  # for sake of simplicity, lets load first 5 accounts.
  @accounts = Account.limit(5).includes(:latest_transactions)
end
```

Turns out, this doesn't work as expected. The queries for the above code were ignoring the limit clause and retrieving all the 
transactions for the accounts. The only thing that worked was the ordering.

Anyways, the limit clause in the above query wasn't going to solve the problem because we needed the limit clause on each 
account's transactions and not all the transactions as a whole. My bad.

So, upon futher searching, I was directed to checkout the _windows function_ provided in Postgres. Here is the 
[documentation](https://www.postgresql.org/docs/9.5/static/functions-window.html) and 
[examples](https://community.modeanalytics.com/sql/tutorial/sql-window-functions/). Please check them out for better clarity.

Using the windows function, we can perform aggregate(sort, order, group, limit etc) on partitioned data. The 3 function that we 
can use in the current situation are _row_number_, _rank_ and _dense_rank_. 

- **row_number**: number of the current row within its partition, counting from 1
- **rank**: rank of the current row with gaps; same as row_number of its first peer. ie, if N records have the same rank X, then the 
  next record in the partition has rank N+X rather than X+1. eg, if you have 5 items at rank 2, the next rank listed would be 
  ranked 7 and not 3.
- **dense_rank**: rank of the current row without gaps; this function counts peer groups. ie, if N records have the same rank X, then 
  the next record in the partition has rank X+1. eg, if you have 5 items at rank 2, the next rank listed would be ranked 3.

We can use any of the 3 functions since the ranking is going to be based on created_at(DateTime) which is always going to be 
unique across all records in the table.

So basically, we would be dividing the transactions collection based on the account_id, order each partition in descending order 
of created_at and ranking each record of the set based on created_at. So, the newest record would have the lowest rank.

So, our basic query is going to be as

```sql
select transactions.*, dense_rank() over(                                                                                         
  partition by transactions.account_id                                                                                            
  order by transactions.created_at desc                                                                                           
) as t_rank                                                                                                             
from transactions;
```

This lists all the transactions partitioned/grouped by transactions.account_id and order in desc order of created_at. Since the 
created_at is going to be unique across all the transactions in the table, the combination of (account_id and t_rank) is unique.

Since we need transactions with t_rank <= 3, we use the above query in a nested select statement.

```sql
select * from (                                                                                                                   
  select transactions.*, dense_rank() over(                                                                                       
    partition by transactions.account_id                                                                                            
    order by transactions.created_at desc                                                                                           
  ) as t_rank                                                                                                           
  from transactions                                                                                                               
) as latest_transactions                                                                                                          
where t_rank <= 3; 
```

Lastly, we don't need the transactions for all accounts but only for specific accounts. In out case, the first 5 accounts.

```sql
select * from (                                                                                                                   
  select transactions.*, dense_rank() over(                                                                                       
    partition by transactions.account_id                                                                                            
    order by transactions.created_at desc                                                                                           
  ) as t_rank                                                                                                           
  from transactions
  where transactions.account_id in (1,2,3,4,5)
) as latest_transactions                                                                                                          
where t_rank <= 3;
```

Now, we have a query that retrieves only the last 3 transactions for specific accounts. Since, we tried using the association with 
includes but couldnt work it out, the only option is to skip includes and load the data explicitly.

So, we declare a class to perform the task. Our class should accept an collection of accounts, retrieve the associated 
transactions using the query defined above and should assign the related transactions to each of the passed accounts.

```ruby
class LatestTransactions
  def initialize(accounts)
    @accounts = accounts
  end

  attr_accessor :accounts

  #generate the inner sql qwery
  def inner_query
    ids = @accounts.collect &:id
    Transaction.select(
      '*, dense_rank() over( 
        partition by transactions.account_id 
        order by transactions.created_at
      ) as t_rank'
    ).where(account_id: ids).to_sql
  end

  #create complete query and execute.
  def transactions
    Transaction.select('*').from("(#{inner}) as latest_transactions").where('t_rank <= 3')
  end
end
```

Finally, we have a service which accepts a collection of Account and fetches their latest transactions. Now, the only problem 
remaining is grouping the transactions as per their account and assigning the group to the particular account. I fixed this by 
adding an attr_accessor to the account model. 

```ruby
class Account
  has_many :transactions
  # has_many :latest_transactions, -> { latest }, class_name: Transaction

  validates :account_number, presence: true

  attr_accessor :latest_transactions
end

class LatestTransactions
  ...

  def associate_accounts_and_transactions
    grouped_transactions = transactions.group_by(&:account_id)
    @accounts.each do |account|
      # account.latest_transactions = transactions.select{|t| t.account_id == account.id}
      account.latest_transactions = grouped_transactions[account.id]
    end
  end
end
```

We can use the above service as

```ruby
def index
  service = LatestTransactions.new Account.limit(5)
  service.associate_accounts_and_transactions
  @accounts = service.accounts
end
```
```haml
# displaying the accounts and their latest 3 transactions
- @accounts.each do |account|
  = display the account's details
  - account.latest_transactions.each do |transaction|
    = each transaction's details
```
The above code fires only 2 queries, first to load the 5 accounts and the second to load the required transactions

```sql
SELECT  "accounts".* FROM "accounts" LIMIT 5
SELECT * FROM (SELECT transactions.*, dense_rank() OVER (
  PARTITION BY transactions.account_id
  ORDER BY transactions.created_at DESC
) AS t_rank FROM "transactions" WHERE "transactions"."account_id" IN (1, 2, 3, 4, 5)) AS latest_transactions WHERE (t_rank <= 3)
```

The above solution is crude and can be definitely improved. Hope you enjoyed it. Critics are welcome.

