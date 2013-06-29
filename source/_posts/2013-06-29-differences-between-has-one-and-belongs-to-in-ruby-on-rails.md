---
layout: post
title: "Differences between has_one and belongs_to in Ruby on Rails"
date: 2013-06-29 16:42
comments: true
author: Neil Rosenstech
categories:
- ruby on rails
---

When getting started with Ruby on Rails, associations between models can become quite confusing especially when there's a thin line between two of them. 
At first we're tempted to use the one that makes more sense when thinking about it... For instance we can say that a *page* belongs to a *book*, but we could say that a 
*page* has one *book* too. The two of them establish a one-to-one association between both models.

But the question we need to ask ourselves here is: **in which model do I want the foreign key to be?** In fact, this is the slight difference between `has_one` and 
`belongs_to` in Ruby on Rails.

**`belongs_to` will place the foreign key in the declaring model whereas `has_one` will place it in the other model.**

Let's see some examples, first using `belongs_to`.

``` ruby
class Book < ActiveRecord::Base
end

class Page < ActiveRecord::Base
  belongs_to :book
end
```

This will use a **book_id** field in the *pages* table (note: of course you need to add that field with a migration). 
It also adds 4 methods in the Page class: `book`, `book=`, `build_book`, `create_book`.

``` ruby
page = Page.create!
=> #<Page id: 1, book_id: nil>

book = Book.create!
=> #<Book id: 1>

page.book = book
=> #<Book id: 1>

page
=> #<Page id: 1, book_id: 1>

other_book = page.create_book!
=> #<Book id: 2>

page
=> #<Page id: 1, book_id: 2>

page.book
=> #<Book id: 2>
```

To make this a one-to-many association just declare the other side of it.

``` ruby
class Book < ActiveRecord::Base
  has_many :pages
end

class Page < ActiveRecord::Base
  belongs_to :book
end
```

Now a book has many pages (and each page still belongs to a book) and you can use the usual methods on the book: 
`pages`, `pages<<`, `pages.find`, `pages.build`, `pages.create` and many more.

If we use a `has_one` association, here what happens:

``` ruby
class Book < ActiveRecord::Base
end

class Page < ActiveRecord::Base
  has_one :book
end
```

Here are some examples:

``` ruby
page = Page.create!
=> #<Page id: 1>

book = Book.create!
=> #<Book id: 1, page_id: nil>

page.book = book
=> #<Book id: 1, page_id: 1>

other_book = page.create_book!
=> #<Book id: 2, page_id: 1>
```

You will probably want to set the other side of the association at some point.

``` ruby
class Book < ActiveRecord::Base
  belongs_to :page
end

class Page < ActiveRecord::Base
  has_one :book
end
```

But that way a book can only have one page...

To sum things up: use `belongs_to` when you want the foreign key in the declaring model, use `has_one` if you want it on the other model.

But anyway, you will rarely see a `belongs_to` or a `has_one` used alone. Most of the time it will be `has_many` with `belongs_to` for a one-to-many 
association and `has_one` with `belongs_to` for a one-to-one association.

