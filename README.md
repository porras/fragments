**NOTE:** This is not real software, maybe it will be one day but it is not today. It is an exercise on [Readme Driven Development](http://tom.preston-werner.com/2010/08/23/readme-driven-development.html), or, put in an alternative way, insane fantasy.

# Fragments

Fragments is a Ruby library to scrape HTML using sample templates. I use it for testing, but for sure it has more uses (basically, anything that implies parsing HTML and treating it as structured data, and that you could do using [Scrapi](https://github.com/assaf/scrapi), [Scrubyt](https://github.com/scrubber/scrubyt), or plain [Nokogiri](http://nokogiri.org/)). Think of it like a template system, the other way around:

    +------+       +-----------+       +------+
    | data | ----> | template  | ----> | HTML |
    +------+       +-----------+       +------+
    
    +------+       +-----------+       +------+
    | HTML | ----> | fragments | ----> | data |
    +------+       +-----------+       +------+

Actually, syntax is <s>very similar to</s> a subset of [Liquid](https://github.com/Shopify/liquid)'s.

Simple example:

    template = Fragments::Template.new(%q{
      <h1>{{title}}</h1>
      <div class="content">
        {{ content }}
      </div>
    })
    
    data = template.parse(%q{
      <html>
      <head>
        <title>Hello, world</title>
      </head>
      <body>
        <h1>Hello world</h1>
        <div class="content">
          Lorem ipsum dolor sit amet...
        </div>
      </body>
      </html>
    })
    
    data.title    # "Hello, world"
    data.content  # "Lorem ipsum dolor sit amet..."

If you expect more than one instance of your fragment, you can use `find` instead of `parse`:

    template = Fragments::Template.new(%q{
      <h1>{{title}}</h1>
      <div class="content">
        {{ content }}
      </div>
    })
    
    data = template.find(%q{
      <html>
      <head>
        <title>My blog</title>
      </head>
      <body>
        <h1>Hello world</h1>
        <div class="content">
          Lorem ipsum dolor sit amet...
        </div>
        <h1>A second article</h1>
        <div class="content">
          Wadus wadus...
        </div>
      </body>
      </html>
    })
    
    data.size        # 2
    data[0].title    # "Hello, world"
    data[0].content  # "Lorem ipsum dolor sit amet..."
    data[1].title    # "A second article"
    data[1].content  # "Wadus wadus..."
    
But in that case you can also simply use Liquid's iterators:

    template = Fragments::Template.new(%q{
      <h1>Products from {{company.name}} </h1>
      <ul id="products">
        {% for product in products %}
          <li>
            <h2>{{ product.name }}</h2>
            Only {{ product.price }}

            {{ product.description }}
          </li>
        {% endfor %}
      </ul>
    })
    
    data = template.parse(%q{
      <html>
      <head>
        <title>Products from The Foo Bar</title>
      </head>
      <body>
        <h1>Products from The Foo Bar</h1>
        <ul id="products">
          <li>
            <h2>Club Mate</h2>
            Only 1.00€
            
            Mate based drink
          </li>
          <li>
            <h2>Berliner</h2>
            Only 2.00€
            
            The classical beer from Berlin
          </li>
        </ul>
      </body>
      </html>
    })
    
    data.company.name             # "The Foo Bar"
    
    data.products.size            # 2
    
    data.products[0].name         # "Club Mate"
    data.products[0].price        # "1.00€"
    data.products[0].description  # "Mate based drink"
    
    data.products[1].name         # "Berliner"
    data.products[1].price        # "2.00€"
    data.products[1].description  # "The classical beer from Berlin"

## RSpec/Capybara matchers

(quick and dirty, not very sure about this API; maybe its simpler to have a helper that returns the scraped response, and do plain method calls and assertions)

Provided you store your fragment templates under `spec/support/fragments/<name>.html.fragment`, you can match against it in your Capybara specs:

    # for using with the second example template
    page.should have_fragment(:product, :count => 2)

    # for using with the third example template
    page.should have_fragment(:company, :company => { :name => "The Foo Bar" },
                                        :products => [ {:name => 'Club Mate', :price => '1.00€'},
                                                       {:name => 'Berliner',  :price => '2.00€'}])

An example of the aforementioned *simpler* approach:

    company = page.as(:company)
    
    company.name.should == "The Foo Bar"
    company.should have(2).products
    company.products[0].name.should == 'Club Mate'
    ...

