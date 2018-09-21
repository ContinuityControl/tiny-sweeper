# TinySweeper

TinySweeper keeps your objects tidy!

![Hold me closer, Tiny Sweeper](https://github.com/ContinuityControl/tiny_sweeper/raw/master/tiny-sweeper.png)

It's a handy way to clean attributes on your Rails models, though it's independent of Rails, and can be used in any Ruby project. It gives you a light-weigt way to override your methods and declare how their inputs should be cleaned.

[![Build Status](https://travis-ci.org/ContinuityControl/tiny_sweeper.png?branch=master)](https://travis-ci.org/ContinuityControl/tiny_sweeper)
[![Code Climate](https://codeclimate.com/github/ContinuityControl/tiny_sweeper/badges/gpa.svg)](https://codeclimate.com/github/ContinuityControl/tiny_sweeper)
[![Test Coverage](https://codeclimate.com/github/ContinuityControl/tiny_sweeper/badges/coverage.svg)](https://codeclimate.com/github/ContinuityControl/tiny_sweeper/coverage)

## How Do I Use It?

```ruby
class Sundae
  attr_accessor :ice_cream

  include TinySweeper
  sweep(:ice_cream) { |flavor| flavor.strip.downcase }
end
```

Now your Sundae toppings will be tidied up:

```ruby
dessert = Sundae.new
dessert.ice_cream = '   CHOCOlate  '
dessert.ice_cream #=> 'chocolate'. Tidy!
```

TinySweeper will not bother you about your nil values; they're your job to handle.

```ruby
Sundae.new.topping = nil  # No topping? TinySweeper won't sweep it.
```

If lots of attributes need to be swept the same way, you can pass an array of field names:

```ruby
class Sundae
  attr_accessor :ice_cream, :topping, :nuts

  include TinySweeper
  sweep [:ice_cream, :topping, :nuts] { |item| item.strip.downcase }
end

dessert = Sundae.new
dessert.ice_cream = '   CHOCOlate  '
dessert.topping = ' ButTTERscotCH   '
dessert.nuts = '  CRUSHED peaNUtS  '
dessert.ice_cream #=> 'chocolate'
dessert.topping #=> 'butterscotch'
dessert.nuts #=> 'crushed peanuts'
```

TinySweeper already knows a few sweeping tricks, and you can ask for them by name:

```ruby
class Sundae
  attr_accessor :ice_cream

  include TinySweeper
  sweep :ice_cream, :blanks_to_nil
end

dessert = Sundae.new
dessert.ice_cream = ""
dessert.ice_cream #=> nil
```

You can use as many as you need, and TinySweeper will apply them all, left-to-right:

```ruby
class Sundae
  attr_accessor :ice_cream

  include TinySweeper
  sweep :ice_cream, :strip, :blanks_to_nil

dessert = Sundae.new
dessert.ice_cream = "   "
dessert.ice_cream #=> nil
end
```

TinySweeper currently only knows a few tricks...

* `blanks_to_nil`: turn empty strings into nils
* `strip`: just like `String#strip`: removes trailing and leading whitespace
* `dumb_quotes`: replace [Smart Quotes](https://en.wikipedia.org/wiki/Quotation_marks_in_English) with their simpler siblings
* `nbsp`: replace [non breaking space characters](https://www.w3.org/MarkUp/HTMLPlus/htmlplus_13.html) with an empty string

...but you can teach it new ones:

```ruby
TinySweeper::Brooms.add(:strip_html) { |value| Nokogiri::HTML(value).text }
```

And you can always combine the built-in tricks with a block:

```ruby
class Sundae
  ...
  sweep(:topping, :strip, :dumb_quotes) { |topping| topping.downcase }
end
```

If you have an object with lots of attributes that need cleaning (because, say, they were loaded from the database), you can do that, too:

```ruby
dessert.sweep_up!
# or:
Sundae.sweep_up!(dessert)
```

### Future Ideas

#### Other Ways to Sweep

Rails models are clearly the natural use-case for this. So it would make sense to have an easy way to auto-clean up models in a table. We'll see. Right now, this works (though it's slow):

```ruby
MyModel.find_each do |m|
  m.sweep_up!
  m.save
end
```

## How Does It Work?

You include the `TinySweeper` module in your class, and define some sweep-up rules on your class' attributes. It prepends an anonymous module to your class, adds to it a method with the same name that cleans its input according to the sweep-up rule, and then passes the cleaned value to `super`.

"Why not use `after_create` or `before_save` or `before_validate` callbacks?"

That's one approach, and it's used by [nilify_blanks](https://github.com/rubiety/nilify_blanks), so it's clearly workable. But it means your data isn't cleaned until the callback runs; TinySweeper cleans your data as soon as it arrives. Also, it requires rails, so you can't use it outside of rails.

## Install It

The standard:

```
$ gem install tiny_sweeper
```

or add to your Gemfile:

```
gem 'tiny_sweeper'
```

## Contributing

Help is always appreciated!

* Fork the repo.
* Make your changes in a topic branch. Don't forget your specs!
* Send a pull request.

Please don't update the .gemspec or VERSION; we'll coordinate that when we release an update.
