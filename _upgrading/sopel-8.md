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

#### Encrypted IRC by default

If its config file doesn't specify otherwise, Sopel 8 assumes that it _should_
`use_ssl` and connect on the standard encrypted `port` for IRC, 6697.

Older versions assumed they should _not_ `use_ssl` and use `port` 6667.

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

The old `fiat_provider` default of `exchangerate.host` is no longer a valid
choice; it has been replaced by `open.er-api.com`.

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

`trigger.time` is now an "aware" `datetime` object, meaning it has a UTC offset
associated with it. Comparison or arithmetic operations between "aware" and
"naive" `datetime` objects are not allowed; code that manipulates `trigger.time`
values will need to be updated for Sopel 8.

The fallback format string used by `sopel.tools.time.format_time()` if no other
source provides one changed from outputting the timezone name (`UTC`) to the UTC
offset (`+0000`).

`format_time()`'s handling of "aware" and "naive" `datetime`s was also improved,
but those changes should be transparent to both users and plugin developers.

`sopel.tools.time.validate_timezone()` now also raises a `ValueError` even if
the timezone to be validated is `None`. This was previously a special case that
_returned_ `None`.

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

#### `STATUSMSG` handling

Events sent to a channel can be scoped to users with a particular privilege
level, or _status_, in that channel. IRC servers advertise the availability of
this feature using the `STATUSMSG` token in `RPL_ISUPPORT`.

In Sopel 7, `trigger.sender` included the status prefix, and it was difficult
for plugins to detect and remove it themselves if so desired. Sopel 8.0 removes
the status prefix from `trigger.sender` and leaves only the channel name, while
the status prefix (if any) is exposed as `trigger.status_prefix`. This change is
useful for plugins that use `trigger.sender` to store or retrieve _channel
values_ in the bot's database, for example.

The `bot` passed to a plugin callable triggered by an IRC event automatically
includes the status prefix, if present, in the default `destination` parameter
to methods that send a _message_ to IRC. Plugin callables that simply invoke
`bot.say()`, `bot.reply()`, etc. in direct response to a `trigger` and without
overriding the default `destination` will most likely get the expected behavior.

[`sopel-remind`][sopel-remind] is an example of a plugin that this change might
surprise, since the naïve implementation of such a reminder plugin would simply
store `(trigger.nick, trigger.sender, parsed_message)` and send
`bot.reply(parsed_message, trigger.sender, trigger.nick)` after the specified
amount of time has passed. The `trigger.status_prefix` is lost, and the reminder
sent to the entire channel, rather than the status-limited subset of users who
could have seen the original command.

#### Only set the `sender` when it makes sense

For general IRC events like `QUIT`, `RPL_NAMREPLY`, etc. that don't happen "in"
a channel or other clearly defined _context_, the `trigger.sender` value that
plugin callables saw in Sopel 7 was unpredictable at best.

Sopel 8.0 now _only_ sets the `trigger.sender` if the triggering event warrants
it. In all other cases, the `sender` will be `None` and plugin code should use
the `trigger.args` list to retrieve all information about the event.

#### To have `text`, you must first have `args`

`trigger.text` will now be _empty_ (`''`) if the event carries no `args`.
In Sopel 7, `trigger.text` contained the command name (e.g. `'QUIT'`) in these
cases, but that was a bug. It has been fixed in Sopel 8.

### IRC privilege requirement changes

In Sopel 8.0, the `@sopel.plugin.require_privilege()` decorator now implies
`@sopel.plugin.require_chanmsg()`, and the decorated callable will not run if
triggered outside of a channel. Previously, `require_privilege` restrictions
were simply ignored, and the callable would run anyway.

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

### The bot's hostmask

In Sopel 8, accessing `bot.hostmask` when the bot lacks sufficient data to
determine its hostmask will return `None`. This replaces the previous behavior,
which was to raise a `KeyError`.

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

* `bot.memory['url_callbacks']` is no longer created or populated by the bot
  * Tracking this information moved to an internal property, in preparation for
    the removal of `bot.register_url_callback()`, `bot.search_url_callbacks()`,
    and `bot.unregister_url_callback()` in Sopel 9.0
  * As a reminder, plugins **should** no longer use the above-mentioned methods;
    the `bot.rules.check_url_callbacks(bot, url)` method now works very well
    with the `@sopel.plugin.url()` decorator
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

More pieces of Sopel's pre-existing API have been deprecated in version 8.0 as
we continue to reorganize and rework various parts of the overall project:

* `bot.search_url_callbacks()` (use `bot.rules.check_url_callbacks(bot, url)`)
* `sopel.tools.OutputRedirect`
* `sopel.tools.stderr()` (we can't stress enough: plugins should use a logger,
  not stdout/stderr output)
