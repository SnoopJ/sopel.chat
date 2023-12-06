---
title: Sopel 8.0 Migration
covers_from: 7.x
covers_to: 8.0
---

# Sopel 8.0 Migration

Version 8.0 brings Sopel into the modern era of Python, finally dropping support
for Python 2 and all end-of-life Python 3 releases as of December 2023. This let
us make significant updates under the hood to future-proof things like the IRC
connection backend and event system.

## Owner/admin usage changes

### Updated Python requirements & support policy

Sopel 8.0 requires Python 3.8 or higher. Support for Python 2.7 and 3.3–3.7 has
been removed. In exchange, known issues sometimes preventing use of Sopel 7 with
Python 3.11+ have been fixed.

During the lifecycle of Sopel 8.x, we will add new Python releases to our test
suite as soon as possible. We will keep testing against end-of-life Python
versions unless and until it becomes technically impossible to do so. However,
if testing against an EOL Python version does become impossible, we will drop it
in the next maintenance release of Sopel.

### CLI changes

Sopel 8 continues the command-line interface overhaul we began in version 7,
mostly in the form of removing support for legacy usage from Sopel's 6.x era.

The bare `sopel` command now offers only basic control. Most of its legacy
options have been removed:

|             Legacy command            |             Modern command             |
|:-------------------------------------:|:--------------------------------------:|
| `sopel --quit` or `sopel -q`          | `sopel stop`                           |
| `sopel --kill` or `sopel -k`          | `sopel stop --kill` or `sopel stop -k` |
| `sopel --restart` or `sopel -r`       | `sopel restart`                        |
| `sopel --configure-all` or `sopel -w` | `sopel configure`                      |
| `sopel --configure-modules`           | `sopel configure --plugins`            |
| `sopel --list` or `sopel -l`          | `sopel-config list`                    |
| `sopel -v`                            | `sopel -V` / `sopel --version`         |
| `sopel --quiet`                       | n/a; this feature didn't work anyway   |

### Ignore system

In Sopel 8, "hostmask" blocks are now called "host" blocks, which reflects the
reality of how those values are used by Sopel. Bot admins' muscle memories will
have to learn to type e.g. `.blocks add host` instead of `.blocks add hostmask`.

We intend to add [actual "hostmask" blocking][better-ignore-system] in a future
Sopel release.

_Note that the **config file** option for "hostmask" blocks already was, and
still is, named `host_blocks`. This change only affects interactive blocklist
editing by bot admins via IRC commands._

  [better-ignore-system]: https://github.com/sopel-irc/sopel/issues/1355

### Configuration & plugin-handling changes

#### What's a "module", doc?

Continuing our push to clarify the difference between a "module" (which is a
Python concept) and a "plugin" (which is a special kind of "module" that can
extend Sopel with new functionality), Sopel 8 no longer loads plugins from the
`<homedir>/modules` directory by default.

#### Taming logging to a channel

Logging is great! Having Sopel output log messages to an IRC channel of your
choice can be very convenient, too. However, it was possible to have Sopel give
you _too much of a good thing_ and get stuck if the `logging_channel_level` was
set to `DEBUG`.

Therefore, `DEBUG` is no longer a valid choice for the `logging_channel_level`
setting, and `logging_channel_level` is no longer inherited from the main
`logging_level` setting. The new default `logging_channel_level` is `WARNING`.

#### Farewell, Phenny/Jenni

The project that eventually became Sopel diverged from Jenni in 2012. After more
than a decade of maintaining backward compatibility (in a theoretical sense, at
least) with plugins originally written for Jenni and its predecessor Phenny,
Sopel 8.0 bids a fond farewell to plugin code from that era.

Beyond the maintenance burden of making sure Sopel could still _technically_
load such old plugins, we felt it was disingenuous to continue "supporting" them
when the rest of Sopel's API has changed and evolved so much that the chance of
such old plugins _actually still working_ is now very low indeed. Thus, we're
not _dropping_ support for those decade-old plugins as much as we're _admitting_
that _things have changed a lot_ and users would be better off seeking (or if
necessary, writing) newer plugins with similar functionality.


## Sopel 8 plugin changes

### Removed built-in plugins

Sopel's built-in plugins are slowly being migrated out to their own standalone
packages, which will help the project manage responsibility for maintenance and
updates in the long term. In 8.0, we say farewell to:

* `help`: now published as [`sopel-help`][sopel-help]
  * Note: Sopel still requires the `sopel-help` package, so it will be installed
    and available automatically.
* `ip`: now published as [`sopel-iplookup`][sopel-iplookup]
* `meetbot`: now published as [`sopel-meetbot`][sopel-meetbot]
* `py`: now published as [`sopel-py`][sopel-py]
* `reddit`: now published as [`sopel-reddit`][sopel-reddit]
* `remind`: now published as [`sopel-remind`][sopel-remind]

  [sopel-help]: https://pypi.org/project/sopel-help/
  [sopel-iplookup]: https://pypi.org/project/sopel-iplookup/
  [sopel-meetbot]: https://pypi.org/project/sopel-meetbot/
  [sopel-py]: https://pypi.org/project/sopel-py/
  [sopel-reddit]: https://pypi.org/project/sopel-reddit/
  [sopel-remind]: https://pypi.org/project/sopel-remind/

### Notable changes to built-in plugins

#### `currency` plugin

The `fiat_provider` setting now takes precedence over the `fixer_io_key`.

Previously, setting a `fixer_io_key` would use the `fixer.io` fiat exchange rate
provider regardless of the `fiat_provider` setting.

#### `reload` plugin

The `.update` command has been removed from the `reload` plugin. Its functioning
relied on running Sopel in a way that isn't officially supported, so we chose to
remove it.

#### `wikipedia` plugin

The old command names (`.w`, `.wik`, and `.wiki`) are no longer used; they were
freed up for use by bespoke plugins. `wikipedia` functionality is now invoked by
the `.wikipedia` command, or `.wp` for short.

The deprecated `lang_per_channel` config-file setting has been removed.


## Sopel 8 API changes

This is a good time to remind you that your plugins can specify both minimum and
maximum Sopel versions in their requirements. If all plugins a bot owner wishes
to install contain this metadata, they can ask `pip` to install Sopel and all of
its plugins in a single command, and `pip`'s dependency solver will figure out
which version of Sopel satisfies all of the plugins' needs.

We do everything we can to keep breaking API changes to *only* major versions,
so a version range like `sopel>=7.1,<9` is typically safe—at least the earliest
Sopel version your plugin supports, up to (but excluding) the next unreleased
major version.

In the vast majority of cases, removed API features will first go through a
deprecation period, during which Sopel will log warnings whenever the deprecated
functionality is used by a plugin. Rarely, we might need to remove an API
feature without going through this process—but that's a last-resort option.

### Command names are now literal

To prevent conflicts with the default capture groups Sopel defines for commands
and prepare for future enhancements to plugin/command management, characters
with special meaning in regular expressions are now escaped in command names.

Documentation for `@sopel.plugin.command()` suggests alternatives based on the
use case you need to adapt for Sopel 8.

### Enums everywhere

_(Insert that classic Buzz Lightyear meme here, if you want.)_

Modernizing Sopel to run only on currently-supported Python versions let us do
quite a bit of work under the hood, but we were also able to start improving how
some parts of the API work. Using `Enum`s where it makes sense is part of that.

The following parts of Sopel's API are now expressed as some type of `Enum`:

* `sopel.formatting.colors`
* `sopel.tools.events`

### Identifier casemapping

Any reasonably modern IRC server advertises its method for normalizing nicknames
to every client that connects. Sopel 8 builds on the `bot.isupport` mechanism
added in Sopel 7 and makes use of the advertised `CASEMAPPING` value when
comparing and normalizing nicknames itself.

To support handling this under the hood, the bot's `config.core.nick` setting is
now stored and returned as a normal `str`. It and any other strings that you'd
like to treat as nicknames should be passed through the `bot.make_identifier()`
method to get the same `Identifier` type you're used to from Sopel 7, configured
to use the IRC server's advertised casemapping method.

### Time-handling changes

The fallback format string used by `sopel.tools.time.format_time()` if no other
source provides one changed from outputting the timezone name (`UTC`) to the UTC
offset (`+0000`).

`format_time()`'s handling of "aware" and "naive" `datetime`s was also improved,
but those changes should be transparent to both users and plugin developers.

### Database changes

`bot.db.get_nick_id()` no longer creates a new ID by default if its `create`
parameter is unspecified.

### IRC event changes

The event names `RPL_INVITELIST` and `RPL_ENDOFINVITELIST` have been updated per
clarifications from the living-specification project at modern.ircdocs.horse.
`RPL_INVEXLIST` and `RPL_ENDOFINVEXLIST` were added to the list of named events
that Sopel knows about.

Plugin code might need to be updated as a result of these changes, but keep in
mind that [different IRC servers mean different things][invitelist-madness] when
they send these events. If in doubt, specify a raw numeric value (e.g. `'346'`).
Writing truly cross-network plugins that react to these events should be
undertaken _very_ carefully.

  [invitelist-madness]: https://github.com/ircdocs/modern-irc/issues/42

### IRC connection status monitoring

Previously a simple `bool` value, `bot.connection_registered` is now a property
combining status information from across the bot's subsystems to answer the very
important question, "Is the `bot` actually registered to an IRC network?" That
is, Sopel not only has a _connection_ open, but the IRC server has _accepted_
the bot as a _client_ and it's possible to _send commands_ to the IRC server.

This is useful to plugins with code that runs without a triggering IRC event but
still outputs to IRC, such as a `@sopel.plugin.interval()` job or acting on data
received [via a socket][sopel-sockmsg]. `if not bot.connection_registered`, the
output can be skipped, retried after a delay, etc.

_Hopefully no one was overwriting the old simple attribute in plugin code, but
now they **can't** do that._

  [sopel-sockmsg]: https://github.com/dgw/sopel-sockmsg

### More consistent `trigger` objects

For general IRC events like `QUIT`, `RPL_NAMREPLY`, etc. that don't happen "in"
a channel or other clearly defined _context_, the value of `trigger.sender` that
plugin callables handling those events would see could be unpredictable at best.

Sopel 8.0 now _only_ sets the `trigger.sender` if the triggering event warrants
it; in all other cases, the `sender` will be `None` and plugin code should use
the `trigger.args` list to retrieve all information about the event.

Additionally, `trigger.text` will now be _empty_ (`''`) if the event carries no
`args`. (`trigger.text` previously contained the command name (e.g. `'QUIT'`) in
these cases, but that was a bug and it has been fixed in Sopel 8.)

### IRC privilege requirement changes

In Sopel 8.0, the `@sopel.plugin.require_privilege()` decorator now implies
`@sopel.plugin.require_chanmsg()`, and the decorated callable will not run if
triggered outside of a channel. Previously, `require_privilege` restrictions
were simply ignored, and the callable would run anyway.

== @TODO: ONLY IF #2580 is merged ==
The `@sopel.plugin.require_bot_privilege()` decorator also now implies
`@sopel.plugin.require_chanmsg()`, with the same associated behavior change.

### Capability negotiation rework

The old `CapReq` method of asking Sopel to negotiate additional capabilities on
behalf of your plugin has been replaced with a much more robust system based on
the new `sopel.plugin.capability` class/decorator.

Plugins can now request capabilities in one of two ways:

```python
"""Sample plugin file"""

from sopel import plugin

# this will register a capability request
CAP_ACCOUNT_TAG = plugin.capability('account-tag')

# this will work as well
@plugin.capability('message-prefix')
def cap_message_prefix(cap_req, bot, acknowledged):
    # do something if message-prefix is ACK or NAK
    ...
```

The first method is easier if you just want to require a capability. The second
is easier if your plugin needs to _do something in response to_ the IRC server's
response to the capability request. See the documentation for more details.

### Testing tool changes

* Convenience methods of the mock IRC server available through Sopel's `pytest`
  plugin implement keyword-only arguments now. This applies to:
  * `MockIRCServer.channel_joined()`
  * `MockIRCServer.mode_set()`
  * `MockIRCServer.join()`
  * `MockIRCServer.say()`
  * `MockIRCServer.pm()`

### Moved API features

We wanted to organize things better in Sopel 8.0, so some of the API features
have moved. The old locations will still work until Sopel 9.0, but you'll be way
ahead of the game if you update your plugins now!

* `Identifier` type moved from `sopel.tools` to `sopel.tools.identifiers`
* `SopelMemory` and `SopelMemoryWithDefault` types moved from `sopel.tools` to
  `sopel.tools.memories`
* `sopel.tools.check_pid()` moved to `sopel.cli.utils`

### Removed API features

Previously deprecated parts of Sopel's API have been removed, including:

* `bot.privileges` channel information (deprecated since Sopel 6.2; replaced by
  `bot.channels`)
* `bot.msg()` method (use `bot.say()`)
* `bot.register()` & `bot.unregister()` methods
  * Pretty much no one should have needed to use these in a plugin. If for some
    reason you did, you'll be able to find the relevant documentation chapter
    that describes their replacements yourself.
* `sopel.cli.utils.redirect_outputs()` function (replaced by standard logging)
* `sopel.config.types.ConfigSection.get_list()` method (use a `ListAttribute`)
* `sopel.irc.utils.CapReq.module` attribute
  * Quick migration: switch to the `plugin` attribute
  * Future-proof migration: use the `sopel.plugin.capability` class; `CapReq`
    will be removed in Sopel 9.0
* `sopel.loader.trim_docstring()` (use `inspect.getdoc()`)
* `sopel.test_tools` (use the `pytest` plugin + mocks/factories)
* `sopel.tools` members of the internal-use kind, deprecated since Sopel 7.1:
  * `compile_rule()`, `get_action_command_pattern()`,
    `get_action_command_regexp()`, `get_command_pattern()`,
    `get_command_regexp()`, `get_nickname_command_pattern()`, &
    `get_nickname_command_regexp()`
* `sopel.tools` members of the useless-in-modern-Python kind:
  * `Ddict` class (use a `collections.defaultdict`)
  * `contains()` methods of `SopelMemory` and `SopelMemoryWithDefault` (use the
    `in` operator)
  * `get_raising_file_and_line()` (use Sopel's logging facility for exceptions)
  * `stdout()` (use Sopel's logging facility, or `print()` if you must)
* `sopel.web` (moved to `sopel.tools.web` in Sopel 7.0)

### Deprecated API features

More pieces of Sopel's pre-existing API have been deprecated as we continue to
reorganize and rework various parts of the overall project:

* `sopel.tools.OutputRedirect`
* `sopel.tools.stderr()` (we can't stress enough: plugins should use a logger,
  not stdout/stderr output)

---- haven't looked past here ----

### Accessing the database

While Sopel's [migration to SQLAlchemy](#database-support) doesn't affect
*most* of [the `bot.db` API][docs-db], some plugins that make use of the more
direct methods might need to be rewritten for Sopel 7. Non-exhaustively:

* [`bot.db.execute()`][docs-db-execute] *should* still return an object that
  behaves like a `Cursor`, but since it's actually a SQLAlchemy wrapper not
  everything is guaranteed to work exactly the same
* [`bot.db.get_uri()`][docs-db-get_uri] hasn't functionally changed, but it's
  important to remember that the returned URI *might* not point to SQLite
* [`bot.db.connect()`][docs-db-connect] is likewise functionally unchanged,
  except that it can now return a non-SQLite DBAPI connection object,
  potentially one with behavior different from the SQLite connections always
  returned in older versions

We recommend that authors of plugins which use a raw database connection from
`bot.db.connect()` aim to rewrite their code so it uses the ORM approach and
[`bot.db.session()`][docs-db-session] instead. In the interim, it never hurts
to update your plugin's documentation to warn users that non-SQLite databases
haven't been tested, or make sure the plugin is marked as compatible with Sopel
<7.0 only until it can be tested and/or updated.

  [docs-db]: /docs/db.html
  [docs-db-connect]: /docs/db.html#sopel.db.SopelDB.connect
  [docs-db-execute]: /docs/db.html#sopel.db.SopelDB.execute
  [docs-db-get_uri]: /docs/db.html#sopel.db.SopelDB.get_uri
  [docs-db-session]: /docs/db.html#sopel.db.SopelDB.session

### Managing URL callbacks

For quite a while, Sopel plugins wishing to override the `url.py` plugin's
automatic title-fetching for certain URLs have customarily done something along
these lines:

```python
# in the plugin's setup() function:
    if not bot.memory['url_callbacks']:
        bot.memory['url_callbacks'] = tools.SopelMemory()
    bot.memory['url_callbacks'][compiled_regex] = methodname
```

Similar manual manipulation of the object in memory was needed to unregister
handlers at plugin unload:

```python
# in the plugin's shutdown() function:
    try:
        del bot.memory['url_callbacks'][compiled_regex]
    except KeyError:
        pass
```

Going forward, a new set of API methods should be used instead:

  - `bot.register_url_callback(pattern, callback)`, to invoke `callback` when a
    URL in a message matches the `pattern`
  - `bot.unregister_url_callback(pattern, callback)`, to stop invoking the
    `callback` when a URL in a message matches the `pattern`
  - `bot.search_url_callbacks(url)`, to find callbacks matching the given `url`

`bot.memory['url_callbacks']` will remain unchanged for the life of Sopel 7.x.
We plan to make this data structure private in Sopel 8.0, so we can improve it
(e.g. allowing multiple callbacks for the same pattern). The new API methods
are already future-proofed against the changes we plan to make; that's why the
callback function is required both when registering *and* unregistering.

### Adding multiple command examples

Decorating a plugin callable like this was a great way to add documentation to
that command through Sopel's `help` plugin:

```python
from sopel import module

@module.example('.foo barbaz')
def foo_cmd(bot, trigger):
    bot.action('foos %s' % trigger.group(3))
```

However, *only one example* could ever appear in the `help` output. This was
[confusing](https://github.com/sopel-irc/sopel/issues/1200) to plugin authors.

Furthermore, if more than one example was defined:

```python
from sopel import module

@module.example('.foo barbaz')
@module.example('.foo spam eggs sausage bacon')
def foo_cmd(bot, trigger):
    bot.action('foos %s' % ', '.join(trigger.group(2).split())
```

It was not necessarily intuitive which example would be displayed if a user
did `.help foo`. The code says it would use `example[0]`. That's the first one
in the list, which makes sense. But take a guess which of the two above would
be used—`.foo barbaz` or `.foo spam eggs sausage bacon`?

Ready to see if you were right?

Sopel would use `.foo spam eggs sausage bacon` as the example, because due to
how decorators work, it ends up first in the internal list despite appearing
last in the source code. Not very intuitive for beginning plugin writers…

So, in Sopel 7, there is a `user_help` argument to `@module.example`. If at
least one of a callable's examples has this attribute set to `True`, all such
examples will be used when outputting help for that command:

```python
from sopel import module

@module.example('.foo barbaz', 'foos barbaz', user_help=True)
@module.example('.foo', "I can't foo that!")
@module.example('.foo egg sausage bacon', 'foos egg, sausage, bacon', user_help=True)
def foo_cmd(bot, trigger):
    if not trigger.group(2):
        return bot.say("I can't foo that!")
    bot.action('foos %s' % ', '.join(trigger.group(2).split())
```

Here, Sopel's help output will show both `.foo barbaz` and `.foo egg sausage
bacon` as examples. Plain `.foo` does not have `user_help=True` (it's purely
there for testing), and so it will not be shown.

Of course, backwards compatibility is important! That's why we used this
approach. Callables without any `user_help=True` examples will behave just
like they would have in Sopel 6 and older: The "first" example (the one
closest to the function's `def` line, last in the source line order) will
appear in `help`'s output.

Making `user_help=True` the default would make *a ton* of sense, definitely!
But if we did that, many (many) existing Sopel (and Willie) plugins would
potentially output "bad" help information—so we elected to keep the old
behavior by default in an effort to minimize any "breakage".

### Logging API rework

Plugins should use the new `sopel.tools.get_logger()` function to get a
logging object, starting with Sopel 7.0. It takes the same argument (the
plugin's name) as its predecessor, `sopel.logger.get_logger()`.

`sopel.logger.get_logger()` will have an extended deprecation cycle, to allow
ample time for the ecosystem to adapt without spamming too many log files at
first. The old function's behavior has been tweaked to work reasonably nicely
with the harmonized logging system implemented for Sopel 7.

Calls to `sopel.logger.get_logger()` will begin emitting deprecation warnings
in Sopel 8.0, to alert stragglers (or users of possibly-abandoned code). We
will remove this function from the API in Sopel 9.0.

### Removal of deprecated attributes

A number of ancient attributes that were considered deprecated many releases
ago _finally_ have been removed, mostly from the `Bot` object. Among them:

  - `bot.ops`
  - `bot.halfplus`
  - `bot.voices`
  - `bot.stats`

This list is not comprehensive, but honestly: If your code breaks because a
removed attribute no longer exists, it's actually been broken for a *long* time
on account of the attributes themselves always being empty. They were just
placeholders to avoid anything raising `AttributeError`. (Sopel's current
maintenance team wouldn't have done it that way, but we can't undo the past.)

### Improvements to testing tools

An absolute *ton* of work went into refactoring how the bot works internally
for Sopel 7, and one side benefit (nay, a goal) of the changes was to make
things more directly testable, without needing to create special "mock"
objects. If you have used `sopel.test_tools` to write unit tests for your
plugin code, you're probably familiar with at least one of `MockConfig`,
`MockSopel`, and `MockSopelWrapper`—three classes Sopel itself used in its own
tests until this release.

With the rewrites for Sopel 7, though, Sopel's tests don't need these mock
classes any more. We test directly on the "real" objects, only switching out
the IRC connection for a fake one that just logs its input and output instead
of actually opening a socket to a remote server. This means that our `Mock*`
classes can be marked as **deprecated**; we will remove them in Sopel 8.

Fortunately, there's also another bit of good news: Sopel now comes with a
`pytest` plugin and a whole set of fixtures, factories, and mocks in
`sopel.tests`. We encourage you to [explore them][docs-test-tools] and update
your plugins' tests (or write them, if you haven't already, you naughty
developer!) to use the new goodies. Trust us—they're *much* nicer to write
tests with than the old tools were!

  [docs-test-tools]: https://sopel.chat/docs/tests.html


## Sopel 7 plugin changes

### Reminder DB migration

Sopel 7 refers to specific instances by config file name whenever possible,
instead of using other pieces of config data such as `nick` or `host` that are
more likely to change. The `remind` and `tell` plugins have been updated with
this in mind, and will attempt to automatically convert their respective data
files to the new name format if the old filename exists.

You probably will not need to do anything. However, if the automatic migration
does fail, it will output (and log) information about what it was trying to
do, and link to this section of the Sopel 7 upgrade guide for convenience.

**If a migration failure brought you here:** Above the link you should find
the old and new filenames the plugin was attempting to use.

**Important:** Ensure the associated instance of Sopel is NOT running
before doing anything. Tampering with the reminder files while Sopel is
running can result in data loss.

In most error cases, migration will fail because the new filename already
exists. The simplest fix is to move or rename the conflicting file, and run
Sopel again so the migration can complete.

If both the old and new files are non-empty, you might want to peek inside
them with a text editor to see what's there before deciding which to keep. The
current format for both plugins' data files is essentially <abbr
title="Tab-Separated Values">TSV</abbr>, and they can be merged by hand
(again, _with Sopel stopped_) if both the old _and_ new files somehow contain
meaningful, unique entries.

And of course, if you need more in-depth assistance with fixing a failed
migration, [our IRC channel][sopel-channel] always welcomes questions.

### Core plugin removals

### `spellcheck`

As of February 2018, the Python bindings for `enchant` [became
unmaintained][pyenchant-unmaintained]. This made installing the `spellcheck`
plugin's dependencies increasingly difficult, and often
[caused][windows-enchant] [problems][linux-enchant] with new installations.

Because of this, the `spellcheck` plugin was rewritten to use `aspell` instead,
and extracted into a [standalone PyPI package][pypi-spellcheck] to eliminate
those non-Python dependencies for installing Sopel itself.

The rewrite also added new commands to manage a custom word list:

  - `.scadd` - stages a word for adding to the bot's word list
  - `.scpending` - lists words pending addition to the bot's custom list
  - `.scdel` - removes a word from the pending list
  - `.scsave` - commits pending words to the bot's word list
  - `.scclear` - clears the list of pending words without saving

Unfortunately, the `aspell` API only supports *adding* words to the custom
dictionary. To *remove* a custom word, a user must manually edit the dictionary
file, so we decided to go with a two-step process. Hopefully it will help Sopel
admins around the world avoid adding typos to their bots' custom dictionaries!

  [pyenchant-unmaintained]: https://github.com/rfk/pyenchant/commit/4df35b7
  [windows-enchant]: https://github.com/sopel-irc/sopel/issues/1142
  [linux-enchant]: https://github.com/sopel-irc/sopel/pull/1454
  [pypi-spellcheck]: https://pypi.org/project/sopel-spellcheck/

### `ipython`

We've made the `ipython` plugin into [its own PyPI package][pypi-ipython],
further reducing the packages required to install Sopel itself. Mostly, though,
this decision was based on the limited utility of this plugin for
non-developers. Its main use case is poking around in Sopel's state while
debugging a new plugin or plugin feature, with Sopel running in an interactive
shell. It's unusable when Sopel is run as a service/daemon (which is how most
deployments *should* run Sopel), and we decided it doesn't make sense to
continue bundling this plugin and its requirements with the core bot.

  [pypi-ipython]: https://pypi.org/project/sopel-ipython/


## Planned user-facing changes

A "user" here is someone who *runs* (or is responsible for *maintaining*) one
or more Sopel bots. If you're reading this, that *probably* describes you.
(Unless you're some kind of weirdo who only writes plugins for other people to
use, and never tests them… Seriously, who does that?)

You might need to edit your bot's configuration in the future due to these
plans. Sopel might take care of them on its own, too. But in case human
intervention is required, here are the details.

### Config format change for list values

Since the new config system was introduced, lists of values have been
represented in the config file as strings separated by commas (`,`). Sopel 7
adds support for storing these lists as multi-line strings instead, with
values separated by newlines.

Here's what that means in practice:

```conf
# Current format
[core]
enable = admin,reddit,wikipedia,wiktionary

# New format
[core]
enable =
    admin
    reddit
    wikipedia
    wiktionary
```

Note that Sopel 7 will upgrade the stored format of any comma-separated list
value if the config is saved (with `.save`) after editing the value via IRC
(with `.set section.option_name list,of,values`).

**Important: The new format requires any value beginning with `#` to be
quoted.** Automatic conversion will handle this, but be aware of it if you're
tweaking your config file(s) by hand during the upgrade process. You will need
to be careful with the `core.channels` list, in particular. Most updated
`core.channels` values should look like this:

```conf
[core]
channels =
    "#spam"
    "#eggs"
    "#bacon"
    "#snakes"
```

Unquoted values beginning with `#` *might* work properly on certain Python
versions *if indented*, but you should quote them anyway to be safe.

In Sopel 7, this is just a convenience change. It means that lists stored in
the new format can support commas within the values, without any annoying
escape-character shenanigans (which was our first plan). Old-style values (all
on a single line) will continue to be split on commas when reading config
files, as before, and will be updated to the new format as described above.

**Eventually, in Sopel 9, we plan to remove this fallback behavior.** Sopel 8
may emit a warning to the console and/or log file at startup if old-style list
values are present in the loaded config file, but we encourage updating your
config files *long* before then.

### Modules vs. plugins

Sopel has long used the term "module" in reference to Python files (or folders
thereof) that add functionality to the bot. This is reflected even in the
layout of Sopel's data folder: `~/.sopel/modules` is so named because that's
where the modules go! We'd like to change that, though.

The simplified explanation is that Python already has "modules". Add-ons for
Sopel are also Python modules, but not all Python modules are Sopel add-ons.
This overlap gets confusing for developers sometimes.

Many similar projects call this kind of add-on a "plugin". We've already
started using the term "plugin" in our documentation, in place of "module",
when referring to these add-ons.

As a user, you'll notice only small changes related to this shift. The
restructured CLI [described above](#cli-restructuring), for example, uses
`sopel configure --plugins` to replace `sopel --configure-modules`. Our
documentation will continue to move toward consistently referring to "plugins"
and "modules" as distinct concepts. We'll also change the default search
location for plugins from `~/.sopel/modules` to `~/.sopel/plugins` in a future
release (probably Sopel 8), but it will be easy to re-add the old folder if
you like via the `core.extra` setting.


## Planned future API changes

This section is all about stuff that won't cause problems *now*, but *will*
break in a future release if not updated. Most of these are planned removals of,
or changes to, API features deprecated long ago.

We suggest reviewing these upcoming changes, and updating your own plugins if
they still use anything listed here, as soon as possible. Updating plugins
published to PyPI should take priority, especially plugins written for Sopel 6
that are not future-proofed by capping Sopel's version in their requirements.

If you use third-party plugins that have not been updated, we encourage you to
inform the author(s) politely that they need to update. Or better yet, submit a
pull request or patch yourself!

### Removal of `bot.privileges`

Sopel 7.x will be the last release series to support the `bot.privileges` data
structure (deprecated in [Sopel 6.2.0][v6.2.0], released January 16, 2016).

Beginning in Sopel 8, `bot.privileges` will be removed and plugins trying to
access it will throw an exception. `bot.channels` will be the _only_ place to
get privilege data going forward.

### Removal of `bot.msg()`

[Back in 6.0][v6.0.0], Sopel's API standardized around a consistent argument
order for messaging functions: `message` first, then an optional `recipient` (or
`destination`, if you like). Part of the old API, `bot.msg()`, has stuck around
since then because it was, [quote][msg-hard-comment], "way too much of a pain to
remove". In fact, it turned out to be quite easy to remove.

None of Sopel's own code uses this old method any more, and we will remove it
entirely in 8.0. Uses of `bot.msg()` in 7.0 will emit a deprecation warning, so
any remaining third-party code that still uses it can be found and patched.

  [msg-hard-comment]: https://git.io/sopel-msg-pain

### Rename/cleanup of `sopel.web`

While the whole `sopel.web` module was marked as deprecated in [Sopel
6.2.0][v6.2.0], because it largely serves as a wrapper around the `requests`
library, parts of it seem to be useful enough that they should be kept around.

For Sopel 8, we intend to move `sopel.web` to `sopel.tools.web`. The new
location is available in Sopel 7 to provide a transitional period. Similar to
how importing from both `willie` and `sopel` worked in the run-up to Sopel 6.0,
it is possible to do any of the following during Sopel 7's life cycle:

  - `import sopel.web`
  - `from sopel import web`
  - `import sopel.tools.web`
  - `from sopel.tools import web`

In Sopel 8, we will remove the pointers from `sopel.web` to the new location.
These explicitly deprecated functions will also be removed at the same time:

  - `sopel.web.get()` — use `requests.get()` directly instead
  - `sopel.web.head()` — use `requests.head()` directly instead
  - `sopel.web.post()` — use `requests.post()` directly instead
  - `sopel.web.get_urllib_object()` — really, just use [`requests`][requests]

We will also tweak the module constants:

  - `sopel.web.default_headers`: renamed to `sopel.tools.web.DEFAULT_HEADERS`
  - `sopel.web.ca_certs`: removed in `sopel.tools.web` — it no longer has any
    function (and was probably not useful for Sopel plugins to import, anyway)

New additions to Sopel's web tools made during the life of 7.x will be
available only in `sopel.tools.web`. Functions and constants that we plan to
remove in Sopel 8 (as listed above) will be available only from the old
`sopel.web` module.

  [requests]: https://pypi.org/project/requests/
  [v6.0.0]: {% link _changelogs/6.0.0.md %}
  [v6.2.0]: {% link _changelogs/6.2.0.md %}

### Finalizing the "plugin" transition

As described [above](#modules-vs-plugins), we're trying to get away from the
ambiguity of calling Sopel add-ons "modules", because that term is already
used by Python itself. All Sopel add-ons are Python modules, but not all
Python modules are Sopel add-ons. Many people in the Sopel community have
tripped over this blurred line, and we have plans to change more than just
documentation eventually.

Already, in Sopel 7, the internal mechanisms for handling add-on code are all
about "plugins" now. The module responsible is named `sopel.plugins`, and its
submodules & members all agree on this nomenclature. But you, a _plugin_
developer, must—confusingly—still import the "_module_" module from `sopel` to
use any of the decorators that make Sopel plugins work. Annoying, isn't it?

Rest assured that we're not done yet. Future releases of Sopel will support
(even encourage) importing `sopel.plugin` instead, which will be the new home
of all plugin-related decorators & constants. `sopel.module` won't go away any
time soon, though. It predates even the name "Willie", after all—we can't just
yank it out without a _long_ deprecation cycle. We might just leave it as a
permanent alias to `sopel.plugin`. (The plan is still [up for
discussion][#1738], if you're interested.)

But, once available, you should _definitely_ use `sopel.plugin` in your new
code. It's the future!

  [#1738]: https://github.com/sopel-irc/sopel/issues/1738
