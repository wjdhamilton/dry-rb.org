---
title: Rails Support
layout: gem-single
---

Rails plugin is implemented in a separate [dry-initializer-rails][dry-initializer-rails] gem.

It provides coercion of assigned values to corresponding ActiveRecord instances.

### Base Example

Add the `:model` setting to `param` or `option`:

```ruby
require 'dry-initializer-rails'

class CreateOrder
  extend Dry::Initializer::Mixin
  extend Dry::Initializer::Rails

  # Params and options
  param  :customer, model: 'Customer' # use either a name
  option :product,  model: Product    # or a class

  def call
    Order.create customer: customer, product: product
  end
end
```

Now you can assign values as pre-initialized model instances:

```ruby
customer = Customer.find(1)
product  = Product.find(2)

order = CreateOrder.new(customer, product: product).call
order.customer # => <Customer @id=1 ...>
order.product  # => <Product @id=2 ...>
```

...or their ids:

```ruby
order = CreateOrder.new(1, product: 2).call
order.customer # => <Customer @id=1 ...>
order.product  # => <Product @id=2 ...>
```

The instance is invoked using method `find_by(id: ...)`.
With wrong ids `nil` values are assigned to corresponding params and options:

```ruby
order = CreateOrder.new(0, product: 0).call
order.customer # => nil
order.product  # => nil
```

### Custom Keys

You can specify custom `key` for searching model instance:

```ruby
require 'dry-initializer-rails'

class CreateOrder
  extend Dry::Initializer::Mixin
  extend Dry::Initializer::Rails

  param  :customer, model: 'User', find_by: 'name'
  option :product,  model: Item,   find_by: :name
end
```

This time you can send names (not ids) to the initializer:

```ruby
order = CreateOrder.new('Andrew', product: 'the_thing_no_123').call

order.customer # => <User @name='Andrew' ...>
order.product  # => <Item @name='the_thing_no_123' ...>
```

### Container Syntax

If you prefer [container syntax][container-syntax], extend plugin inside the block:

```ruby
require 'dry-initializer-rails'

class CreateOrder
  include Dry::Initializer.define -> do
    extend Dry::Initializer::Rails

    # ... params/options declarations
  end
end
```

### Types vs Models

[Type constraints][type-constraints] are checked before the coercion.

When mixing `:type` and `:model` settings for the same param/option, you should use [sum types][sum-types] that accept both model instances and their attributes.

[container-syntax]: http://dry-rb.org/gems/dry-initializer/container-version/
[dry-initializer-rails]: https://github.com/nepalez/dry-initializer-rails
[sum-types]: http://dry-rb.org/gems/dry-types/sum/
[type-constraints]: http://dry-rb.org/gems/dry-initializer/type-constraints/
