# Pirata
[![Gem Version](https://badge.fury.io/rb/pirata.svg)](http://badge.fury.io/rb/pirata) [![Pirata API Documentation](https://www.omniref.com/ruby/gems/pirata.png)](https://www.omniref.com/ruby/gems/pirata)

**Note: I'm available for consulting and contract software development work. If you're
looking to build a website, API, automation or scraping services then get in touch: [colin@strongco.de](mailto:colin@strongco.de)**

Pirata is a Ruby gem that exposes a useful and easy to use API for the popular
torrent tracker [ThePirateBay](http://thepiratebay.se). It aims to give developers
a simple way to incorporate the torrent and website data into their applications.

# Usage
## Quick Start Guide
**Note** As the `nokogiri` gem is a runtime dependency of this one, it is highly
recommended you install that before hand like so:

    $ env NOKOGIRI_USE_SYSTEM_LIBRARIES=true gem install nokogiri

The reason for this is that `nokogiri` attempts to compile libxml2 and libxslt
at installation. Substituting already-installed system libraries will cut down
drastically on the installation time. This is not required, but hopefully will be helpful.

### Quick Start

First, download pirata with `gem install pirata`

Next, require it (either in IRB or your .rb file) `require 'pirata'`

Now you are free to play with Pirata! Here are some examples
```ruby
# A basic search across all categories
search = Pirata::Search.new("zelda reorchestrated") # => Return a Pirata::Search object
search.results # => An array of Torrent objects resulting from the search
search.results.first # => A Pirata::Torrent object. See the below table for available methods

# Get an array of the top 100 Torrents
top_hundred = Pirata::Search.top

# Now within a specified category
top_hundred_videos = Pirata::Search.top(Pirata::Category::VIDEO)

# Get an array of the 30 most recent uploaded Torrents
most_recent = Pirata::Search.recent

# You can also find detailed information on a Torrent if you know its ID
Pirata::Torrent.find_by_id(5241636)
```

## Advanced Searching
```ruby
# Order results by Seeders
torrents = Pirata::Search.new("open source", Pirata::Sort::SEEDERS).results

# Order results by Date, search only in Video and Applications categories
torrents = Pirata::Search.new("open source", Pirata::Sort::DATE, [Pirata::Category::VIDEO, Pirata::Category::APPLICATIONS]).results

# Check if a search is multipage
query = Pirata::Search.new("open source").multipage? # => true

# Find the last page number
query.pages # => 3

# Perform a search on a page beyond the first
query.search_page(2)
```

## Torrent Objects
`torrent = Pirata::Search.new("zelda").results.first`

| Method     | Result               | Example value                                   | Return Type |
|------------|----------------------|-------------------------------------------------|-------------|
| #title     | Torrent title        | "Cylum's 'The Legend of Zelda' ROM Collection"  | String      |
| #category  | Torrent category     | "Games"                                         | String      |
| #url       | Full URL             | "http://thepiratebay.si/torrent/10080116/Cyl..."| String      |
| #id        | Numeric ID           | 10080116                                        | Fixnum      |
| #magnet    | Torrent magnet link  | "magnet:?xt=urn:btih:30f784d135af21152052a..."  | String      |
| #seeders   | Number of seeders    | 4                                               | Fixnum      |
| #leechers  | Number of leechers   | 0                                               | Fixnum      |
| #uploader  | User who uploaded torrent | #<Pirata::User:0x0000000260f200>           | Pirata::User|

Note: The following methods require an extra request to be made, but the first request (regardless of
which method you call) will fetch and populate data for all other calls for the same Torrent object.

| Method     | Result               | Example value                                   | Return Type |
|------------|----------------------|-------------------------------------------------|-------------|
| #files     | Number of files      | 51                                              | Fixnum      |
| #size      | Total torrent size   | "224.95 MiB (235876575 Bytes)"                  | String      |
| #comments  | Number of comments   | 0                                               | Fixnum      |
| #hash      | Alphanumeric hash    | "30F784D135AF21152052A45AE718A7FCAB597A79"      | String      |
| #date      | Date of upload       | 2002-01-01 00:00:00 -0500                       | Time        |

## User Objects

| Method     | Result               | Example value                                   | Return Type |
|------------|----------------------|-------------------------------------------------|-------------|
| #username  | Username string      | "LXZX"                                          | String      |
| #profile_url | URL for the user profile | "http://thepiratebay.si/user/LXZX"        | String      |

# Categories
Searches by default will query across all categories. However you can choose to narrow your search down
by passing an array of categories. These are all namespaced under Pirata::Category. **Please note:** These
are only the main/topmost categories for searching - there are subcategories for each category listed below
(but the list is too large to be pasted here). Please reference `lib/pirata/category.rb` for an entire
list of possible search categories.

```ruby
    AUDIO         = "100"
    VIDEO         = "200"
    APPLICATIONS  = "300"
    GAMES         = "400"
    PORN          = "500"
    OTHER         = "600"
```

# Sorting
By default, results are sorted by ThePirateBay's relevance algorithm. While you could manually sort Torrent
objects by writing your own comparator, you can also pass a `Pirata::Sort` constant to your search and
have the results returned to you in whichever sorting fashion you choose.
```ruby
    RELEVANCE   = "99"
    TYPE        = "13"
    NAME        = "1"
    DATE        = "3"
    SIZE        = "5"
    UPLOADER    = "11"
    SEEDERS     = "7"
    LEECHERS    = "9"
```

# Config
The root `Pirata` namespace has an instance method `Pirata.configure` which accepts an options
hash. There are 2 values accepted - `:base_url` and `:redirect`. The former refers to the
mirror site you wish to use. ThePirataBay is notorious for going down and coming back up under
a new domain. Because of this, you may have to change the domain you are querying against if
you find you are running into trouble. Luckily, there are a lot of great people who host TPB
mirror/proxy sites for free! If you find you are getting timeout errors, you can set the base
url to a working proxy.

A list of mirror sites can be found at [TheProxyBay](http://proxybay.info/)

There is also a rule for HTTP redirection. When specifying a protocol in the `:base_url` value, you may choose to
use `http` or `https`. The `open_uri_redirections` patch allows HTTP <-> HTTPS redirections, but requires a rule
to be specified. You can use `:safe` or `:all`. The former only does HTTP -> HTTPS redirection, while the latter
will do both. By default, it is set to `:all`.

The default configuration looks like this:
```ruby
@config = {
  :base_url => 'https://pirateproxy.tv/',
  :redirect => :all
}
```

and can be overridden with `Pirata.configure({:base_url => 'http://someproxy.com', :redirect => :safe})`

The configuration can be accessed with `Pirata.config`, which will return a hash. Note that all keys are symbols.

# Testing
There is a basic test suite you can run that will test against basic use cases and functionaliy. The big problem
with testing this gem is that 1) the data is always changing for search results as things get uploaded or removed,
comments get added, leechers/seeders get added or dropped off, etc, and 2) ThePirateBay could go offline. You can
run tests by simply running `rake` in the parent Pirata directory. If you find you are getting errors (but not
failures), ensure that the domain you are using in `Pirata.config[:base_url]` is actually up. Otherwise, please
feel free to submit a Github issue and I'll get to it asap. Or fork it and help out!

# License
This program is free software: you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
