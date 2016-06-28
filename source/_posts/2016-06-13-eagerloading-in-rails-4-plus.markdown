---
layout: post
title: "Eagerloading in Rails 4+"
date: 2016-06-13 23:24:17 +0530
comments: true
categories: 
---

User has_many Address
Address belongs_to User

Account has_many Transaction

User.includes(:addresses) 2 queries
User.preload(:addresses) 2 queries
User.eager_load(:addresses) 1 query, left outer join, distinct users
User.joins(:addresses) 1 query, inner join, duplicate users

Rails 4.2:
references:
no need to use with eager_load or joins
need to be used with includes and preload

User.includes(:addresses).where("addresses.country = ?", "Poland") V/S User.includes(:addresses).where(addresses: {country: 
'Poland'})

Queries
### 1
users = User.includes(:addresses).where("addresses.country = ?", "Poland") # 0 records
users = User.preload(:addresses).where("addresses.country = ?", "Poland") # 0 records
users = User.eager_load(:addresses).where("addresses.country = ?", "Poland") # 1 record
users.first.addresses # 1 record

users = User.includes(:addresses).where("addresses.country = ?", "Poland").references(:addresses) #1 record
users.first.addresses # 1 record
users = User.preload(:addresses).where("addresses.country = ?", "Poland").references(:addresses) #0 records
users = User.eager_load(:addresses).where("addresses.country = ?", "Poland").references(:addresses) #1 record
users.first.addresses # 1 record

users = User.includes(:addresses).where(addresses: {country: 'Poland'}) # 1 record
users.first.addresses # 1 record
users = User.preload(:addresses).where(addresses: {country: 'Poland'}) # 0 records
users = User.eager_load(:addresses).where(addresses: {country: 'Poland'}) # 1 record
users.first.addresses # 1 record

### 2
Get all users with all addresses with atleast one being polish
users = User.joins(:addresses).where("addresses.country = ?", "Poland")
users.first.addresses # 2 records

### 3
Get users with atleast one address
users = User.joins(:addresses)
3 records, 2 duplicates
users = User.joins(:addresses).distinct
2 records, 0 duplicates

### 4
Get all users with no addresses
users = User.includes(:addresses).where(addresses: {user_id: nil})

Accounts and Transactions

-List last 3 transactions for all accounts
-List last 3 debit transactions for all accounts
-List last 3 credit transactions for all accounts
-List last 3 transactions on a particular date
-List last 3 credit transactions greater than a particular amount
-List last 3 debit transactions smaller than a particular amount
-List account with no transactions

select accounts.*, 
dense_rank() over (
partition by transactions.account_id
order by transactions.created_at desc
) as transaction_rank
from accounts left outer join transactions on accounts.id = transactions.account_id order by accounts.id;

select id, number, created_at, updated_at from ( select accounts.*, 
dense_rank() over (
partition by transactions.account_id
order by transactions.created_at desc
) as transaction_rank
from accounts left outer join transactions on accounts.id = transactions.account_id order by accounts.id) as ranked_transactions 
where transaction_rank < 4;

I was working on a simple feature where in I wanted to list the parent models. Againt each parent model record, its latest N 
children were to to be displayed. Lets consider a sample application for simplicity.

We are to display a list of accounts(Account). Against each account, we need to list the latest 3 transactions.

Firstly, I included the transactions when loading the accounts and then decided to restrict the transactions count for each 
account.

def index
  #for sake of simplicity, lets load transactions for first 5 accounts.
  @accounts = Account.limit(5).includes(:transactions)
  # SELECT  "accounts".* FROM "accounts" LIMIT 3
  # SELECT "transactions".* FROM "transactions" WHERE "transactions"."account_id" IN (1, 2, 3)
end

# displaying the accounts
- @accounts.each do |act|
  -#display the account details
  - account.transactions.order(created_at :desc).limit(3)

This seemed to work but wasn't. Its because a query was fired for loading the last 3 transactions for each account leading 
to N+1 queries.
  
  # SELECT  "transactions".* FROM "transactions" WHERE "transactions"."account_id" = $1  ORDER BY "transactions"."created_at" DESC 
  LIMIT 3  [["account_id", 1]]

I had read somewhere that we can define relations as scopes. Since we can order/filter the data in the scope, I decided to preload 
the new association. It was worth a try.

So,

class Transaction
  belongs_to :account

  scope :latest, -> { order(created_at: :desc).limit(3) }
end

class Account
  has_many :transactions
  has_many :latest_transactions, -> { latest }, class_name: Transaction
end

Confident that this will work, I modified the original code to get the last 3 transactions for first 5 accounts.

def index
  @accounts = Account.limit(5).includes(:latest_transactions)
end

Atlast, this didnt work too. The queries for the above code were ignoring the limit clause and retrieving all the transactions for 
the accounts.

ie, the query being fired was as

SELECT "transactions".* FROM "transactions" WHERE "transactions"."account_id" IN (1, 2, 3)  ORDER BY "transactions"."created_at"  
DESC

Anyways, the limit clause in the above query isnt going to solve the problem because we need the limit clause on each account's 
transactions and not all the transactions as a whole. My bad.

So, upon futher googling, i stumbled upon the windows function https://www.postgresql.org/docs/9.5/static/functions-window.html
 and https://community.modeanalytics.com/sql/tutorial/sql-window-functions/

So, we write our query as

select transactions.*, dense_rank() over(
  partition by transactions.account_id
  order by transactions.created_at desc
) as transaction_rank
from transactions;

this is going to list all the transactions grouped by transactions.account_id and order in desc order of created_at. Since the 
created_at is going to be unique across all the transactions in the table, the combination if (account_id and transaction_rank) is 
going to be unique too.

So we select only the transactions where the transaction_rank is <= 3.

select * from (
  select transactions.*, dense_rank() over(
  partition by transactions.account_id
  order by transactions.created_at desc
  ) as transaction_rank
  from transactions
) as ranked_transactions
where transaction_rank <= 3;

Since we need not load the last 3 transactions for ALL the accounts but only selected records, we need to filter the transactions 
based on transactions.account_id.

So, we modify the above query as

select * from(
 select transactions.*, dense_rank() over(
   partition by transactions.account_id
   order by transactions.created_at desc
 ) as transaction_rank
 from transactions
 where transactions.account_id in (1,2,3)
) as ranked_transactions
where transaction_rank <= 3;

Now, we have the query that retrieves only the last 3 transactions for specific accounts. Since, we tried using the relation with 
includes but couldnt work it out, IMO, the only option is to skip includes and load the data explicitly.

So, we declare a service to load the latest transactions for each account.

class LatestTransactions
  def initialize(accounts)
    @accounts = accounts
  end

  def inner_query
  =begin
    select transactions.*, dense_rank() over(                                                                                        
      partition by transactions.account_id                                                                                           
      order by transactions.created_at desc                                                                                          
    ) as transaction_rank                                                                                                            
    from transactions                                                                                                                
    where transactions.account_id in (1,2,3)
  =end
  =begin
    Transaction.select('*').where(account_id: Account.limit(3).pluck(:id)).to_sql
      SELECT  "accounts"."id" FROM "accounts" LIMIT 3
      "SELECT * FROM \"transactions\" WHERE \"transactions\".\"account_id\" IN (1, 2, 3)" 
    Transaction.select('screw u').where(account_id: Account.limit(3).pluck(:id)).to_sql
      SELECT  "accounts"."id" FROM "accounts" LIMIT 3
      "SELECT screw u FROM \"transactions\" WHERE \"transactions\".\"account_id\" IN (1, 2, 3)" 

    Transaction.select('*, dense_rank() over( partition by transactions.account_id order by transactions.created_at) as 
    transaction_rank').where(account_id: Account.limit(3).pluck(:id)).to_sql
      SELECT  "accounts"."id" FROM "accounts" LIMIT 3
      "SELECT *, dense_rank() over( partition by transactions.account_id order by transactions.created_at) as transaction_rank 
      FROM \"transactions\" WHERE \"transactions\".\"account_id\" IN (1, 2, 3)" 
  =end

  #since we dont need to fire query to load the account's id and that we only need the sql query and not the actual list of 
  transactions, we use to_sql.
    ids = @accounts.collect &:id
    Transaction.select('*, dense_rank() over( partition by transactions.account_id order by transactions.created_at) as 
    transaction_rank').where(account_id: ids).to_sql
  end

  def transactions
    =begin
      Transaction.select('*').from("something").to_sql
      "SELECT * FROM something"
    =end
    Transaction.select('*').from("(#{inner}) as latest_transactions").where('transaction_rank <= 3')
    SELECT * FROM (SELECT *, dense_rank() over( partition by transactions.account_id order by transactions.created_at) as 
    transaction_rank FROM "transactions" WHERE "transactions"."account_id" IN (1, 2, 3)) as latest_transactions WHERE 
    (transaction_rank <= 3)
  end
end

now, we have a service which accepts a collection of Account and fetches its latest transactions. Now, the only problem remaining 
is grouping the transactions as per their account and assigning the group to the particular account. This i solved by adding an 
attr_accessor to the account model. 

class LatestTransactions
  def associate_accounts_and_transactions
    grouped_transactions = transactions.group_by(&:account_id)
    @accounts.each do |account|
      account.latest_trans = grouped_transactions[account.id]
      account.latest_trans = transactions.select{|t| t.account_id == account.id}
    end
  end
end

The above solution is crude and can be definitely improved.

