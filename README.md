# counter_culture

Turbo-charged counter caches for your Rails app. Huge improvements over the Rails standard counter caches:

* Updates counter cache when values change, not just when creating and destroying 
* Supports counter caches through multiple levels of relations
* Supports dynamic column names, making it possible to split up the counter cache for different types of objects
* Executes counter updates after the commit, avoiding [deadlocks](http://mina.naguib.ca/blog/2010/11/22/postgresql-foreign-key-deadlocks.html)

## Installation

Add counter_culture to your Gemfile:

```ruby
gem 'counter_culture', '~> 0.1.4'
```

Then run ```bundle update ```

## Database Schema

You will need to manually create the necessary columns for all counter caches. Use code like the following in your migration:

```ruby
add_column :categories, :products_count, :integer, :null => false, :default => 0
```

It is important to make the column ```NOT NULL``` and set a default of zero for this gem to work correctly.

## Usage

### Simple counter-cache

```ruby
class Product < ActiveRecord::Base
  belongs_to :category
  counter_culture :category
end

class Category < ActiveRecord::Base
  has_many :products
end
```

Now, the ```Category``` model will keep an up-to-date counter-cache in the ```products_count``` column of the ```categories``` table.

### Multi-level counter-cache

```ruby
class Product < ActiveRecord::Base
  belongs_to :sub_category
  counter_culture [:sub_category, :category]
end

class SubCategory < ActiveRecord::Base
  has_many :products
  belongs_to :category
end

class SubCategory < ActiveRecord::Base
  has_many :sub_categories
end
```

Now, the ```Category``` model will keep an up-to-date counter-cache in the ```products_count``` column of the ```categories``` table. This will work with any number of levels.

### Customizing the column name

```ruby
class Product < ActiveRecord::Base
  belongs_to :category
  counter_culture :category, :column_name => "products_counter_cache"
end

class Category < ActiveRecord::Base
  has_many :products
end
```

Now, the ```Category``` model will keep an up-to-date counter-cache in the ```products_counter_cache``` column of the ```categories``` table. This will also work with multi-level counter caches.

### Dynamic column name

```ruby
class Product < ActiveRecord::Base
  belongs_to :category
  counter_culture :category, :column_name => Proc.new {|model| "#{model.product_type}_count" }
  # attribute product_type may be one of ['awesome', 'sucky']
end

class Category < ActiveRecord::Base
  has_many :products
end
```

Now, the ```Category``` model will keep two up-to-date counter-caches in the ```awesome_count``` and ```sucky_count``` columns of the ```categories``` table. Products with type ```'awesome'``` will affect only the ```awesome_count```, while products with type ```'sucky'``` will affect only the ```sucky_count```. This will also work with multi-level counter caches.

### Dynamically over-writing affected foreign keys

```ruby
class Product < ActiveRecord::Base
  belongs_to :category
  counter_culture :category, :foreign_key_values => 
      Proc.new {|category_id| [category_id, Category.find_by_id(category_id).try(:parent_category).try(:id)] }
end

class Category < ActiveRecord::Base
  belongs_to :parent_category, :class_name => 'Category', :foreign_key => 'parent_id'
  has_many :children, :class_name => 'Category', :foreign_key => 'parent_id'
  
  has_many :products
end
```

Now, the ```Category``` model will keep an up-to-date counter-cache in the ```products_count``` column of the ```categories``` table. Each product will affect the counts of both its immediate category and that category's parent. This will work with any number of levels.

## Contributing to counter_culture
 
* Check out the latest master to make sure the feature hasn't been implemented or the bug hasn't been fixed yet.
* Check out the issue tracker to make sure someone already hasn't requested it and/or contributed it.
* Fork the project.
* Start a feature/bugfix branch.
* Commit and push until you are happy with your contribution.
* Make sure to add tests for it. This is important so I don't break it in a future version unintentionally.
* Please try not to mess with the Rakefile, version, or history. If you want to have your own version, or is otherwise necessary, that is fine, but please isolate to its own commit so I can cherry-pick around it.

## Copyright

Copyright (c) 2012 BestVendor. See LICENSE.txt for further details.
