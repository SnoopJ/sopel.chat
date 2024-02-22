---
title: Sopel 8.0 Migration
covers_from: 7.x
covers_to: 8.0
---

# Sopel 8.0 Migration

<!-- NOTE: reworded and added link to official docs with Gantt chart/table of lifecycle info -->

Version 8.0 brings Sopel into the modern era of Python, finally dropping support
for [end-of-life](python-devguide-versions) Python releases as of December 2023,
including Python 2 and several older versions of Python 3. This allows us to make significant improvements under the
hood that were held back by these legacy versions, e.g. the IRC connection
backend and Sopel's event system.

[python-devguide-versions]: https://devguide.python.org/versions/

## Owner/admin usage changes

### Updated Python requirements & support policy

Sopel 8.0 requires Python 3.8 or higher. Support for Python 2.7 and 3.3–3.7 has
been removed. In exchange, known issues <!-- TODO: this is vague --> sometimes preventing use of Sopel 7 with
Python 3.11+ have been fixed.

<!--
TODO: this feels like it might be making stronger promises than we want

I think what we want to say is that we are prioritizing releases that are not
EOL, and we will adopt a best-effort approach to support for out of date
versions. It seems important to me to be explicit in reserving the right to
drop an EOL version because of Sopel maintenance concerns. In particular, the
language "technically impossible" strikes me as *far* too strong and inviting
technical debate from only-slightly-hypothetical users who are upset about
dropping support this way.

In other words, I think we want to avoiding making promises here, but it would
be nice to reword this to state our general philosophy: we want to track new
Python versions as quickly as possible, and we want to be flexible about
supporting legacy versions, UNTIL it becomes a pain in the ass for us to do so. Future versions get priority over old versions?

It is probably also worth mentioning that CPython is now on an 18-month (IIRC? There's a PEP about it) release cycle, which means that we can MUCH more reliably predict new versions and plan ahead for evaluation of pre-releases. If we're going to make forward-looking promises here, saying that we plan to start testing when the first alpha/beta/rc of a new version is out is a good idea. Personally I would advocate for testing alphas, but we could probably get away with betas. Waiting for an RC is probably suboptimal, because if there's a bug in CPython that our CI would unearth, we're screwed after beta is over and will have to skip that release entirely.
-->
During the lifecycle of Sopel 8.x, we will add new Python releases to our test
suite as soon as possible. We will keep testing against end-of-life Python
versions unless and until it becomes technically impossible to do so. However,
if testing against an EOL Python version does become impossible, we will drop it
in the next maintenance release of Sopel.

### CLI changes

Sopel 8 continues the command-line interface overhaul we began in version 7,
mostly in the form of removing support for legacy usage from Sopel's 6.x era.

The bare `sopel` command now offers only basic control. Most of its legacy
options have been removed. For migration purposes, the table below lists removed
commands and their modern equivalents:

<!-- NOTE: I believe I'm correct in saying that the left-hand have been removed, see 39811b72 -->
|             Legacy command (removed)  |             Modern command             |
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
reality of how those values are used by Sopel <!-- TODO: this could be worded more clearly -->. Bot admins' muscle memories will
have to learn to type e.g. `.blocks add host` instead of `.blocks add hostmask`.

We intend to add [actual "hostmask" blocking][better-ignore-system] in a future
Sopel release.

_Note that the **config file** option for "hostmask" blocks already was, and
still is, named `host_blocks`. This change only affects interactive blocklist
editing by bot admins via IRC commands._

  [better-ignore-system]: https://github.com/sopel-irc/sopel/issues/1355

### Configuration & plugin-handling changes

#### What's a "module", doc?

<!-- NOTE: I reversed the clauses here, the migration guide reader cares about the change more than the rationale. -->
Sopel 8 no longer loads plugins from the `<homedir>/modules` directory by default. To migrate, rename
this directory to `plugins/` or declare it in the `extra` field of the `[core]` configuration section.
<!-- TODO: link to config -->

This is part of our continuing our push <!-- TODO: issue link? --> to strengthen the distinction between a "module" (which is a
[Python concept](python-glossary-module)) and a "plugin" (Sopel's terminology for a special kind of "module" that can extend the bot with new functionality).

<!-- TODO: I guessed this URL since I'm editing without network access, need to check it before submitting -->
[python-glossary-module]: https://docs.python.org/3/glossary.html#term-module

#### Encrypted IRC by default

Sopel 8 enables SSL and uses port 6697 by default, breaking with the old behavior
of no SSL and port 6667 by default. Users who want to opt out of SSL should set
the `use_ssl` and `port` options in their bot's config file in the `[core]` section.

#### Taming logging to a channel

<!-- TODO: Put the thing the reader cares about _first_ -->

Logging is great! Having Sopel output log messages to an IRC channel of your
choice can be very convenient, too. However, it was possible to have Sopel give
you _too much of a good thing_ and get stuck if the `logging_channel_level` was
set to `DEBUG`.

Therefore, `DEBUG` is no longer a valid choice for the `logging_channel_level`
setting, and `logging_channel_level` is no longer inherited from the main
`logging_level` setting. The new default `logging_channel_level` is `WARNING`.

#### Farewell, Phenny/Jenni

<!--
TODO: 'what' and 'why' are inverted here as well. What about this section is
actionable for "migration"? Is our advice effectively "stop doing that", i.e.
port your plugins (we don't have guidance for doing that AFAIK) or remove/rewrite
them?
-->

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

In Sopel 8.0, the following plugins have been removed from the core and extracted
to separate packages:

* `help`: now published as [`sopel-help`][sopel-help]
  * Note: Sopel still requires the `sopel-help` package, so it will be installed
    and available automatically.
* `ip`: now published as [`sopel-iplookup`][sopel-iplookup]
* `meetbot`: now published as [`sopel-meetbot`][sopel-meetbot]
* `py`: now published as [`sopel-py`][sopel-py]
* `reddit`: now published as [`sopel-reddit`][sopel-reddit]
* `remind`: now published as [`sopel-remind`][sopel-remind]

Sopel's built-in plugins are slowly being migrated out to their own standalone
packages <!-- TODO: issue link? -->, which will help the project manage responsibility
for maintenance and updates in the long term. 

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
remove it.  <!-- TODO: should we advise affected users to `.reload`, restart Sopel, or neither? -->

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
<!--
TODO: last sentence needs to be clarified

- Need to clarify that plugins can therefore *change the Sopel version* the
user has installed. If users care about that, Sopel must always be a part of
the install command and it should probably be pinned.

- Claim that pip will determine the Sopel version is misleading, pip will find
whatever best solves _the entire graph_, which *might* include an 'old' version
of a plugin if other plugins introduce inflexible constraints on Sopel's version.

-->

We do everything we can to keep breaking API changes to *only* major versions,
so a version range like `sopel>=7.1,<9` <!-- TODO: why not `sopel>=8,<9` since this is the 8.0 migration guide? I think we need to clarify here that the user should determine whatever makes sense for their plugin, and they can use our soft promise about breaking API changes to guide that choice. --> is typically safe—at least the earliest
Sopel version your plugin supports, up to (but excluding) the next unreleased
major version.

In the vast majority of cases, removed API features will first go through a
deprecation period, during which Sopel will log warnings whenever the deprecated
functionality is used by a plugin. Rarely, we might need to remove an API
feature without going through this process—but that's a last-resort option.

<!-- TODO: Should we mention the "breaking change" label on GitHub? -->
<!-- NOTE: overall, this section feels like general guidance on plugin authorship, rather than 8.0 migration advice. It definitely does not provide any advice about what API has changed in 8.0 which might annoy a reader that came to this guide for that information. -->

### Command names are now literal

Characters <!-- TODO: which ones? --> with special meaning in regular expressions
are now escaped in command names.

This prevents conflicts with the default capture groups Sopel defines for commands
and prepare for future enhancements to plugin/command management, characters
with special meaning in regular expressions are now escaped in command names.

Documentation for `@sopel.plugin.command()` <!-- TODO: link it --> suggests
alternatives that can be used to migrate affected commands to Sopel 8.

### Enums everywhere

_(Insert that classic Buzz Lightyear meme here, if you want.)_

The following parts of Sopel's API are now expressed as some type of `Enum`:

* `sopel.formatting.colors`
* `sopel.tools.events`

Modernizing Sopel to run only on currently-supported Python versions let us do
quite a bit of work under the hood, but we were also able to start improving how
some parts of the API work. Using `Enum`s where it makes sense is part of that.

### Identifier casemapping

The bot's `config.core.nick` setting is now stored and returned as a normal `str`.
Use `bot.make_identifier()` to convert this and any other string values to an
`Identifier` that uses the correct case mapping for the current server.

This change allows Sopel 8 to better support any nick normalization that may 
be performed by the IRC server. Any reasonably modern IRC server advertises its
method for normalizing nicknames to every client that connects. Sopel 8 builds
on the `bot.isupport` mechanism added in Sopel 7 and makes use of the advertised
`CASEMAPPING` <!-- TODO: spec link appropriate here? --> value when comparing and normalizing nicknames itself.

### Time-handling changes

<!-- TODO: this section feels like it needs a top-level summary? I can't think
of what to write other than "we made improvements to timezone handling" -->

`trigger.time` is now an "aware" `datetime` object <!-- TODO: Python docs link -->, meaning it has a UTC offset
<!-- TODO: is that more correct than saying "timezone"? IIRC, not all timezones are defined by UTC offsets (sigh) --> associated with it. Comparison or arithmetic operations between "aware" and
"naive" `datetime` objects are not allowed; code that manipulates `trigger.time`
values will need to be updated for Sopel 8. <!-- TODO: *how*? This guide's raison d'être is to guide the reader -->

The fallback format string used by `sopel.tools.time.format_time()` if no other
source provides one changed from outputting the timezone name (`UTC`) to the UTC
offset (`+0000`). <!-- TODO: woof, this is hard to read even as a native speaker; reword -->

`format_time()`'s handling of "aware" and "naive" `datetime`s was also improved,
but those changes should be transparent to both users and plugin developers. <!-- TODO: should this guide even mention it, then? if so, probably need to clarify why they need to know about it here vs. the changelog -->

`sopel.tools.time.validate_timezone(None)` now raises a `ValueError`, removing a
special case that _returned_ `None` in Sopel 7.

### Database changes

`bot.db.get_nick_id()` no longer creates a new ID by default if its `create`
parameter is unspecified.

### IRC event changes

The event names `RPL_INVITELIST` and `RPL_ENDOFINVITELIST` have been updated <!-- TODO: we should be more clear about what changed. what were the old names? --> per
clarifications from the living-specification project at modern.ircdocs.horse <!-- TODO: make this a link to the specific section we're talking about here -->.
`RPL_INVEXLIST` and `RPL_ENDOFINVEXLIST` were added to the list of named events
that Sopel knows about.

Plugin code that handles these events might need to be updated as a result of
these changes, but keep in mind that [different IRC servers mean different things][invitelist-madness]
when they send these events. If in doubt, specify a raw numeric value (e.g. `'346'`).
Writing truly cross-network plugins that react to these events should be
undertaken _very_ carefully.

  [invitelist-madness]: https://github.com/ircdocs/modern-irc/issues/42

### IRC connection status monitoring

`bot.connection_registered` now indicates that a _connection_ is open, the 
IRC server has _accepted_ the bot as a client, and commands can be sent.
Previously, this boolean value indicated only that a connection was open. This
attribute is now read-only and cannot be modified.

<!-- TODO: did I reword that correctly? -->

This is useful to plugins with code that runs without a triggering IRC event but
still outputs to IRC, such as a `@sopel.plugin.interval()` job or acting on data
received [via a socket][sopel-sockmsg]. `if not bot.connection_registered`, the
output can be skipped, retried after a delay, etc.
<!--
TODO: this last sentence needs clarification, I *think* it's saying something like:

"Plugins that want to send commands to the IRC server can write … and skip or retry
if a full connection is not available."
-->

  [sopel-sockmsg]: https://github.com/dgw/sopel-sockmsg

### Changes to `Trigger` objects

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

Sopel 8.0 now _only_ sets `trigger.sender` for the following events:
  * `INVITE`
  * `JOIN`
  * `KICK`
  * `MODE`
  * `NOTICE`
  * `PART`
  * `PRIVMSG`
  * `TOPIC`

<!-- TODO: Do we have a linkable doc section about `COMMANDS_WITH_CONTEXT`? See 4e7ab24f -->

In all other cases, the `sender` will be `None` and plugin code <!-- TODO: ALL plugin code, or a subset of plugins that care about this distinction? --> should use the `trigger.args` list to retrieve all information about the event.

For general IRC events like `QUIT`, `RPL_NAMREPLY`, etc. that don't happen "in"
a channel or other clearly defined _context_, the `trigger.sender` value that
plugin callables saw in Sopel 7 was unpredictable at best.

#### To have `text`, you must first have `args`

`trigger.text` will now be _empty_ (`''`) if the event carries no `args`.
In Sopel 7, `trigger.text` contained the command name (e.g. `'QUIT'`) in these
cases, but that was a bug. <!-- TODO: link the issue? --> This has been fixed in Sopel 8. 

### IRC privilege requirement changes

In Sopel 8.0, the `@sopel.plugin.require_privilege()` decorator now implies
`@sopel.plugin.require_chanmsg()`, and the decorated callable will not run if
triggered outside of a channel. Previously, `require_privilege` restrictions
were simply ignored in a private message, and the callable would run anyway.

<!-- NOTE: Hmm, I wish we called these decorators "channel privilege", but oh well. Possible naming clarification looking forward to 9.x? -->
The `@sopel.plugin.require_bot_privilege()` decorator also now implies
`@sopel.plugin.require_chanmsg()`, with the same associated behavior change.

### Capability negotiation rework

Capability negotation now uses `sopel.plugin.capability`, replacing the old `CapReq`
method.

Plugins can now request capabilities in one of two ways:

#### Simple capability requirement

If a plugin only needs to declare that it depends on a capability, then
`plugin.capability` can be used at the top-level of the plugin: 

```python
"""Sample plugin file"""

from sopel import plugin

# registers a request for the 'account-tag' capability
CAP_ACCOUNT_TAG = plugin.capability('account-tag')
```

<!-- TODO: what happens when the capability is NAKed? Does `CAP_ACCOUNT_TAG` have a further use in this context? -->

#### Fine-grained control over capability negotiation

If your plugin needs to _do something in response to_ the IRC server's
response to the capability request. See the documentation <!-- TODO: link --> for more details.

<!-- TODO: Do I understand right that graceful degradation of a plugin in the absence requires this approach? -->

```python
"""Sample plugin file"""

from sopel import plugin

@plugin.capability('message-prefix')
def cap_message_prefix(cap_req, bot, acknowledged):
    # do something if message-prefix is ACK or NAK
    ...
```

### The bot's hostmask

In Sopel 8, accessing `bot.hostmask` now returns `None` when the bot lacks sufficient data to
determine its hostmask. This replaces the previous behavior, which was to raise a `KeyError`.

### Testing tool changes

* The following testing convenience methods of the mock IRC server now accept the `blocking` parameter
_only_ by keyword argument: <!-- TODO: do we intend to introduce future parameters that way? am I missing something or is it really just this one parameter that is affected? -->
  * `MockIRCServer.channel_joined()`
  * `MockIRCServer.mode_set()`
  * `MockIRCServer.join()`
  * `MockIRCServer.say()`
  * `MockIRCServer.pm()`

### Moved API features

* `Identifier` type has been moved from `sopel.tools` to `sopel.tools.identifiers`
* `SopelMemory` and `SopelMemoryWithDefault` types have been moved from `sopel.tools` to
  `sopel.tools.memories`
* `sopel.tools.check_pid()` have been moved to `sopel.cli.utils`

The old locations will still work until Sopel 9 (when they will be removed),
but we encourage plugin authors to start using the new locations now.

### Removal of previously-deprecated API

The following API features have been removed in Sopel 8:

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

### New API deprecations

The following API features have been deprecated in 8.0 as we continue to
reorganize and rework the overall project.

#### Scheduled for removal in Sopel 8.1

* `sopel.tools.OutputRedirect`
* `sopel.tools.stderr()` (we can't stress enough: plugins should use a logger,
  not stdout/stderr output)

#### Scheduled for removal in Sopel 9.0

* `bot.search_url_callbacks()` (use `bot.rules.check_url_callbacks(bot, url)`)
