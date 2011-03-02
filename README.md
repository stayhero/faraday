# faraday

Modular HTTP client library using middleware heavily inspired by Rack.

This mess is gonna get raw, like sushi. So, haters to the left.

## Usage

    conn = Faraday.new(:url => 'http://sushi.com') do |builder|
      builder.use Faraday::Request::UrlEncoded  # convert request params as "www-form-urlencoded"
      builder.use Faraday::Request::JSON        # encode request params as json
      builder.use Faraday::Adapter::Logger      # log the request to STDOUT
      builder.use Faraday::Adapter::NetHttp     # make http requests with Net::HTTP

      # or, use shortcuts:
      builder.request  :url_encoded
      builder.request  :json
      builder.adapter  :logger
      builder.adapter  :net_http
    end
    
    ## GET ##

    response = conn.get '/nigiri/sake.json'     # GET http://sushi.com/nigiri/sake.json
    response.body

    conn.get '/nigiri', 'X-Awesome' => true     # custom request header
    
    conn.get do |req|                           # GET http://sushi.com/search?page=2&limit=100
      req.url '/search', :page => 2
      req.params['limit'] = 100
    end
    
    ## POST ##
    
    conn.post '/nigiri', { :name => 'Maguro' }  # POST "name=maguro" to http://sushi.com/nigiri
    
    # post payload as JSON instead of "www-form-urlencoded" encoding:
    conn.post '/nigiri', payload, 'Content-Type' => 'application/json'

    # a more verbose way:
    conn.post do |req|
      req.url '/nigiri'
      req.headers['Content-Type'] = 'application/json'
      req.body = { :name => 'Unagi' }
    end

If you're ready to roll with just the bare minimum:

    # default stack (net/http), no extra middleware:
    response = Faraday.get 'http://sushi.com/nigiri/sake.json'

## Testing

    # It's possible to define stubbed request outside a test adapter block.
    stubs = Faraday::Adapter::Test::Stubs.new do |stub|
      stub.get('/tamago') { [200, {}, 'egg'] }
    end

    # You can pass stubbed request to the test adapter or define them in a block
    # or a combination of the two.
    test = Faraday.new do |builder|
      builder.adapter :test, stubs do |stub|
        stub.get('/ebi') {[ 200, {}, 'shrimp' ]}
      end
    end

    # It's also possible to stub additional requests after the connection has
    # been initialized. This is useful for testing.
    stubs.get('/uni') {[ 200, {}, 'urchin' ]}

    resp = test.get '/tamago'
    resp.body # => 'egg'
    resp = test.get '/ebi'
    resp.body # => 'shrimp'
    resp = test.get '/uni'
    resp.body # => 'urchin'
    resp = test.get '/else' #=> raises "no such stub" error

    # If you like, you can treat your stubs as mocks by verifying that all of 
    # the stubbed calls were made. NOTE that this feature is still fairly
    # experimental: It will not verify the order or count of any stub, only that
    # it was called once during the course of the test.
    stubs.verify_stubbed_calls

## TODO

* support streaming requests/responses
* better stubbing API
* Support timeouts
* Add curb, em-http, fast_http

## Note on Patches/Pull Requests

* Fork the project.
* Make your feature addition or bug fix.
* Add tests for it. This is important so I don't break it in a
  future version unintentionally.
* Commit, do not mess with rakefile, version, or history.
  (if you want to have your own version, that is fine but bump version in a commit by itself I can ignore when I pull)
* Send me a pull request. Bonus points for topic branches.

## Copyright

Copyright (c) 2009-2011 rick, hobson. See LICENSE for details.
