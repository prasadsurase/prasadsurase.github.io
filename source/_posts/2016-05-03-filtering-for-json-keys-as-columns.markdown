---
layout: post
title: "Filtering for json keys as columns in active admin"
date: 2016-05-03 23:01:07 +0530
categories: [activeadmin, json, postgresql]
external-url: "http://prasadsurase.github.io/blog/2016/04/25/filtering-for-json-keys-as-columns/"
url: "http://prasadsurase.github.io/blog/2016/04/25/filtering-for-json-keys-as-columns/"
comments: true
published: true
---

After implementing sorting on keys of a json column data, I need to add filtering too. We are using 
ActiveAdmin(1.0.0-pre1) with postgres 9.4. You can check my 
[previous](http://prasadsurase.github.io/blog/2016/04/25/sorting-for-json-keys-as-columns/) post regarding sorting. 

For sake of continuity, lets take the same example as in the previous post. We have a model named Box which 
stores (length, width & height) as json in a column named dimensions.

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

In ActiveAdmin, the filters can be displayed as below

```ruby
ActiveAdmin.register Box do
  filter :length, as: :numeric, label: 'Length'
  filter :breadth, as: :numeric, label: 'Breadth'
  filter :height, as: :numeric, label: 'Height'
  filter :volume, as: :numeric, label: 'Volume'

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

Since some of the fields are not associated with the table, we need need to define custom ransackers for them. 
ActiveAdmin uses ransack gem for implementing searching.

```ruby
class Box < ActiveRecord::Base
  ransacker :length do |parent|
    # '->>' is a part of the syntax for accessing the json keys, 'dimensions' the column name and 
    # 'length' is the actual key on which the search is to be implemented.
    Arel::Nodes::InfixOperation.new('->>', parent.table[:dimensions], Arel::Nodes.build_quoted('length'))
  end
  
  ransacker :breadth do |parent|
    Arel::Nodes::InfixOperation.new('->>', parent.table[:dimensions], Arel::Nodes.build_quoted('breadth'))
  end
  
  ransacker :height do |parent|
    Arel::Nodes::InfixOperation.new('->>', parent.table[:dimensions], Arel::Nodes.build_quoted('height'))
  end
  

  # The below ransackers do not work. Also, we need to declare three ransackers :length_equals, :length_greater_than & 
  # :length_less_than for each field. Dont know the reason but am looking into it.

  ransacker :length_greater_than, formatter: -> (value){
    data = where("dimensions ->> 'length' > '#{value}'").pluck :id
    data = data.present? ? data : nil
  } do |parent|
    parent.table[:id]
  end
end
```

This still won't work. The filters are displayed weirdly(dropdown on top and search input field at the bottom) rather in 
the expected layout(dropdown on left and search input field on the right). This is because ActiveAdmin adds a class 
[*select_and_search*](https://github.com/activeadmin/activeadmin/blob/master/lib/active_admin/inputs/filters/base/search_method_select.rb#L34) which decides if a filter field is searchable or not. This class is responsible for the proper 
layout and working of filters. The below code is referenced from ActiveAdmin. It decides whether the class is to be 
added or not when the filter is initialized.

```ruby
module ActiveAdmin
  module Inputs
    module Filters
      module Base
        module SearchMethodSelect
          def wrapper_html_options
            opts = super
              (opts[:class] ||= '') << ' select_and_search' unless seems_searchable?
            opts
          end
        end
      end
    end
  end
end

module ActiveAdmin                                                                                                                                        
  module Filters
    module FormtasticAddons
      def seems_searchable?
        # ransacker? checks if any custom ransackers are defined. 
        has_predicate? || ransacker? || scope?
      end
    end
  end
end
```

So, we remove the check for *ransacker?* by adding a patch in the initializers.

```ruby
# config/initializers/hack_for_filter.rb

module ActiveAdmin                                                                                                                                        
  module Filters
    module FormtasticAddons
      def seems_searchable?
        has_predicate? || scope?
      end
    end
  end
end
```

Now, the filters will be displayed properly and would send the selected option(Greater than, Equals or Less than) with 
the specified value to the ransackers.

I am not inclined towards overriding the library methods but haven't found any other solution for the same. So, for now, 
this it the hack that works for me.

Working source code can be found [here](https://github.com/prasadsurase/active_admin_with_json_field). Hope it was interesting.
