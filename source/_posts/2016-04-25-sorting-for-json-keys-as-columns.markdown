---
layout: post
title: "Sorting for json keys as column in active admin"
date: 2016-04-25 23:01:50 +0530
categories: [activeadmin, json, postgresql]
external-url: http://prasadsurase.github.io/blog/2016/04/25/sorting-for-json-keys-as-columns/
published: true
comments: true
---

Recently, I had a requirement wherein I need to have sorting on the keys of a json column data. We were using postgres 9.4 and the 
schema of the json data was static for all the records. The keys needed to be displayed as columns with sorting enabled against them. 
Below is the way I implemented it. Also, on digging into ActiveAdmin, I learned some interesting implementation details of the library.

Lets take an example. We have a model named Box which stores (length, width & height) as json in a column named dimensions.

```sql
\d boxes;

Table "public.boxes"
Column   |            Type             |                     Modifiers                      
------------+-----------------------------+----------------------------------------------------
id         | integer                     | not null default nextval('boxes_id_seq'::regclass)
user_id    | integer                     | 
dimensions | json                        | 
volume     | integer                     | 
created_at | timestamp without time zone | not null
updated_at | timestamp without time zone | not null
Indexes:
"boxes_pkey" PRIMARY KEY, btree (id)


# select * from boxes limit 5;
  id | user_id |             dimensions              | volume |         created_at         |         updated_at         
 ----+---------+-------------------------------------+--------+----------------------------+----------------------------
   1 |       3 | {"length":4,"breadth":7,"height":3} |     84 | 2016-04-19 18:24:54.281761 | 2016-04-19 18:30:21.885756
   2 |       3 | {"length":3,"breadth":3,"height":8} |     72 | 2016-04-19 18:24:54.28487  | 2016-04-19 18:30:21.904715
   3 |       2 | {"length":1,"breadth":4,"height":3} |     12 | 2016-04-19 18:24:54.287235 | 2016-04-19 18:30:21.911726
   4 |       4 | {"length":0,"breadth":2,"height":4} |      0 | 2016-04-19 18:24:54.289777 | 2016-04-19 18:30:21.917753
   5 |       4 | {"length":7,"breadth":4,"height":0} |      0 | 2016-04-19 18:24:54.292116 | 2016-04-19 18:30:21.924538
(5 rows)
```

In ActiveAdmin, columns can be displayed for non-table fields. All you have to do is project them in the query. In the below example, 
length, breadth and height are not columns of boxes but have been projected in the select query. 


```ruby
ActiveAdmin.register Box do
  controller do
    def scoped_collection
      Box.select("*, dimensions ->> 'length' as length, dimensions ->> 'breadth' as breadth, dimensions ->> 'height' as height")
    end
  end

  index do
    column :length, label: 'Length', sortable: "dimensions->>'length'"
    column :breadth, label: 'Breadth', sortable: "dimensions->>'breadth'"
    column :height, label: 'Height', sortable: "dimensions->>'height'"
    column :volume, label: 'Volume', sortable: true
  end
end
```

When defining the columns to be displayed, we pass the query by which the json key would be accessed when sorting is applied. 
By default ActiveAdmin appends *_asc* and *_desc* to the table columns when sorting. Here, we pass the query overriding the default column name. 

ActiveAdmin has a method named [apply_sorting](https://github.com/activeadmin/activeadmin/blob/1c85c5654a2ce1d43d4c64d98b928ff133d46406/lib/active_admin/resource_controller/data_access.rb#L210) 
which check if the order is valid or not and calls the ordering query.

```ruby
# When sorting the breadth in descending order, the params sent to the controller are as below.
# {"order"=>"dimensions->>'length'_desc", "controller"=>"admin/boxes", "action"=>"index"}

def apply_sorting(chain)
  params[:order] ||= active_admin_config.sort_order #default order for sorting which is 'id_desc'

  orders = []
    params[:order].split('_and_').each do |fragment|
    order_clause = ActiveAdmin::OrderClause.new(fragment) #dimensions->>'breadth'_desc
    # <ActiveAdmin::OrderClause:0x007fad6dfe @column="dimensions->>'length'", @field="dimensions->>'length'", @op=nil, @order="desc">
    if order_clause.valid? #check if field.present? and order.present?
      orders << order_clause.to_sql(active_admin_config)
    end
  end

  if orders.empty?
    chain
  else
    chain.reorder(orders.shift).order(orders) 
    # executes SELECT  *, dimensions ->> 'length' as length, dimensions ->> 'breadth' as breadth, dimensions ->> 'height' as height FROM "boxes"  ORDER BY "boxes".dimensions->>'length' desc;
  end
end
```

Hope it was interesting.
