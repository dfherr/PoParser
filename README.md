# PoParser

[![Build Status](https://travis-ci.org/arashm/PoParser.svg?branch=master)](https://travis-ci.org/arashm/PoParser)
[![Coverage Status](https://img.shields.io/coveralls/arashm/PoParser.svg)](https://coveralls.io/r/arashm/PoParser)
[![Code Climate](https://codeclimate.com/github/arashm/PoParser.png)](https://codeclimate.com/github/arashm/PoParser)
[![Gem Version](https://badge.fury.io/rb/PoParser.svg)](http://badge.fury.io/rb/PoParser)

A Ruby PO file parser, editor and generator. PO files are translation files generated by GNU/Gettext tool. This GEM is compatible with [GNU PO file specification](https://www.gnu.org/software/gettext/manual/html_node/PO-Files.html). report misbehaviours and bugs, to the [issue tracker](https://github.com/arashm/PoParser/issues).

## Installation

Add this line to your application's Gemfile:

    gem 'PoParser'

And then execute:

    $ bundle install

Or install it yourself as:

    $ gem install PoParser

## Usage

Working with the GEM is pretty easy:

```ruby
require 'poparser'
path = Pathname.new('example.po')
po   = PoParser.parse(path)
=> <PoParser::Po, Translated: 68.1% Untranslated: 20.4% Fuzzy: 11.5%>
```

The `parse` method returns a `PO` object which contains all `Entries`:

```ruby
# get all entries
po.entries # or .all alias

# including cached/obsolete entries (started with "#~")
# more info: https://www.gnu.org/software/gettext/manual/html_node/Obsolete-Entries.html
po.entries(true)

# get all fuzzy entries
po.fuzzy

# get all untranslated entries
po.untranslated

# get all translated entries
po.translated

# get all cached/obsolete entries
po.cached # or .obsolete alias

# returns a hash representation of the PO file
po.to_h

# returns a string representation of the PO file
po.to_s
```

You can add a new entry to the PO file:

```ruby
new_entry = {
              translator_comment: 'comment',
              reference: 'reference comment',
              msgid: 'untranslated',
              msgstr: 'translated string'
            }

po.add(new_entry)

# There's also an alias for add `<<`
po << new_entry
```

You can pass an array of hashes to `new_entry` and it will be added to `PO` file.

Adding plural string is the same:

```ruby
new_entry = {
              translator_comment: 'comment',
              reference: 'reference comment',
              msgid_plural: 'untranslated',
              'msgstr[0]': 'translated string',
              'msgstr[1]': 'translated string'
            }

```

Note: currently `PoParser` won't warn you if you add a `msgstr[0]` without `msgid_plural`, any `msgid` at all or even if index numbers in `msgstr[0]` are not in any logical order.

### Entry

Each entry can have following properties (for more information see [GNU PO file specification](https://www.gnu.org/software/gettext/manual/html_node/PO-Files.html)):

```
translator_comment
reference
extracted_comment
flag
previous_msgctxt
previous_msgid
previous_msgid_plural
cached # obsolete entries
msgid
msgid_plural
msgstr
msgctxt
```

#### Working with entries

The `PO` object contains many `Entry` objects. Number of methods are available to check state of the `Entry`:

```ruby
entry = po.entries[1]

entry.untranslated? # or .incomplete? alias
#=> false
entry.translated? # or .complete? alias
#=> true
entry.fuzzy?
#=> true
entry.plural?
#=> false
entry.cached? # or .obsolete?
#=> false
```

You can get or edit each property of the `Entry`:

```ruby
entry.msgid.to_s
#=> "This is an msgid that needs to get translated"
entry.translate("This entry is translated") # or `msgstr=` alias
entry.msgstr.to_s
#=> "This entry is translated"
```

But be careful with plural messages, there msgstr is an array
```ruby
if entry.plural?
  entry.msgstr.each do |msgstr|
    msgstr.to_s
    #=> This is one of the plural translations
  end  
end  
```

You can mark an entry as fuzzy:

```ruby
entry.flag_as_fuzzy
entry.fuzzy?
#=> true
```

It's possible to get Hash and String representation of the `Entry`:

```ruby
entry.to_h
entry.to_s
```

### Searching

`PO` is an `Enumerable`. All exciting methods from `Enumerable` are available in `PO`. The `PO` yields `Entry`.

```ruby
po.find_all do |entry|
  entry.msgid.str.match(/some text/i)
end
```

There is a helper method that does the same:

```ruby
po.search_in(:msgstr, 'some string to search')
```

This method returns an array of matched entries.

### Saving
You can simply save the PO file using the `PO` object:

```ruby
po.save_file
```

If you want to save as the file in diffrent location change the `path`:

```ruby
po.path
#=> example.po
po.path = 'example2.po'
po.save_file
```

## Header

The first entry of every PO file is reserved for header which represent license and general information about translators and the po file itself. For more information visit the [GNU website](https://www.gnu.org/software/gettext/manual/html_node/Header-Entry.html#Header-Entry).

Get header with `header`!:

```ruby
po.header
```

You can get and set following variables from `header`:

```
  pot_creation_date
  po_revision_date
  project_id
  report_to
  last_translator
  team
  language
  charset
  encoding
  plural_forms
```

##To-Do

* Streaming support
* Update header after changing/saving po
* add `before_save` and `after_save` callbacks

## Contributing

1. Fork it ( http://github.com/arashm/poparser/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request
