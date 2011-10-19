ZNC cmd notify
=============

ZNC cmd notify  is a module for [ZNC][] that will send notifications to a command that you
can customize to invoke any action you want (Forward email/SMS messages) for any private 
message or channel highlight that matches a configurable set of conditions.

This project is forked from ZNC to Notifo created by [John Reese](http://johnmreese.com).

Minor edits made by Benwa.


Compiling
---------

If you have installed ZNC from a Linux distribution's repository, you will most likely
need to install the development package before building this module.  On Ubuntu, this can
be installed with:

    $ sudo aptitude install znc-dev

If you have `make` installed, you can compile the module with:

    $ make

Otherwise, run the full command:

    $ znc-buildmod cmdnotify.cpp


Installation
------------

Create the notify script at /etc/notify.sh that takes
in a message as parameters. Example:

    #!/bin/bash
    echo "$@" | mail ...@msg.telus.com
    EOF
    
Copy the compiled module into your ZNC profile:

    $ cp cmdnotify.so ~/.znc/modules/

Now, load the module in ZNC:

    /msg *status loadmod cmdnotify

Commands
--------

*   `help`

    Links you to this fine document.

*   `set <option> <value>`

    Allows you to modify configuration values.

*   `append <option> <value>`

    Allows you to add a string to end of a configuration value.  Automatically adds a
    space to separate the appended value from the existing value.

*   `prepend <option> <value>`

    Allows you to add a string to beginning of a configuration value.  Automatically adds
    a space to separate the prepended value from the existing value.

*   `get [<option>]`

    Allows you to see current configuration values.

*   `unset <option>`

    Allows you to reset a configuration option back to the default value.

*   `status [<context>]`

    Check the status of current conditions.  Specifying the "context" of either a channel
    or nick name will provide status values specific to that context.

*   `send <message>`

    Manually trigger a notification with the given message.  Useful for testing to validate
    credentials, etc.

*   `eval <expression>`

    Evaluate the given expression in an empty context.  Useful for testing to validate that
    a given expression is properly formatted and does not contain invalid tokens.


Configuration
-------------

### Settings

*   `cmd = "/home/user/znc_notify.sh"`

    The command to run when a notification is triggered. Defaults to `/etc/znc_notify.sh`.

### Conditions

*   `away_only = "no"`

    If set to "yes", notifications will only be sent if the user has set their `/away` status.

    This condition requires version 0.090 of ZNC to operate, and will be disabled when
    compiled against older versions.

*   `client_count_less_than = 1`

    Notifications will only be sent if the number of connected IRC clients is less than this
    value.  A value of 0 (zero) will disable this condition.

*   `highlight = ""`

    Space-separated list of highlight strings to match against channel messages using
    case-insensitive, wildcard matching.    Strings will be compared in order they appear in the configuration value, and
    the first string to match will end the search, meaning that earlier strings take priority
    over later values.

    Individual strings may be prefixed with:

    *   `-` (hypen) to negate the match, which makes the string act as a filter rather than
        a search

    *   `_` (underscore) to trigger a "whole-word" match, where it must be surrounded by
        whitespace to match the value

    *   `*` (asterisk) to match highlight strings that start with any of the above prefixes

    As an example, a highlight value of "-pinto car" will trigger notification on the
    message "I like cars", but will prevent notifications for "My favorite car is the Pinto"
    *and* "I like pinto beans".  Conversely, a highlight value of "car -pinto" will trigger
    notifications for the first two messages, and only prevent notification of the last one.

    As another example, a value of "_car" will trigger notification for the message "my car
    is awesome", but will not match the message "I like cars".

*   `idle = 0`

    Time in seconds since the last activity by the user on any channel or query window,
    including joins, parts, messages, and actions.  Notifications will only be sent if the
    elapsed time is greater than this value.  A value of 0 (zero) will disable this condition.

*   `last_active = 0`

    Time in seconds since the last message sent by the user on that channel or query window.
    Notifications will only be sent if the elapsed time is greater than this value.  A value
    of 0 (zero) will disable this condition.

    Note that this condition keeps track of the last message sent to each channel and query
    window separately, so a recent PM to Joe will not affect a notification sent from
    channel #foo.

*   `last_notification = 0`

    Time in seconds since the last notification sent from that channel or query window.
    Notifications will only be sent if the elapsed time is greater than this value.  A value
    of 0 (zero) will disable this condition.

    Note that this condition keeps track of the last notification sent from each channel and
    query window separately, so a recent PM from Joe will not affect a notification sent
    from channel #foo.

*   `nick_blacklist = ""`

    Space-separated list of nicks.  Applies to both channel mentions and query windows.
    Notifications will only be sent for messages from nicks that are not present in this
    list, using a case-insensitive comparison.

    Note that wildcard patterns can be used to match multiple nicks with a single blacklist
    entry. For example, `set nick_blacklist *bot` will not send notifications from nicks
    like "channelbot", "FooBot", or "Robot".  Care must be used to not accidentally
    blacklist legitimate nicks with wildcards.

*   `replied = "no"`

    If set to "yes", notifications will only be sent if you have replied to the channel or
    query window more recently than the last time a notification was sent for that context.


### Notifications

*   `message_length = 1024`

    Maximum length of the notification message to be sent.  The message will be nicely
    truncated and ellipsized at or before this length is reached.  A value of 0 (zero) will
    disable this option.

### Advanced

*   `channel_conditions = "all"`

    This option allows customization of the boolean logic used to determine how conditional
    values are used to filter notifications for channel messages.  It evaluates as a full
    boolean logic expression,  including the use of sub-expressions.  The default value of
    "all" will bypass this evaluation and simply require all conditions to be true.

    The expression consists of space-separated tokens in the following grammar:

    *   expression = expression operator expression | "(" expression ")" | value
    *   operator = "and" | "or"
    *   value = "true" | "false" | condition
    *   condition = <any condition option>

    As a simple example, to replicate the default "all" value, would be the value of
    "away_only and client_count_less_than and highlight and idle and last_active and
    last_notification and nick_blacklist and replied".

    Alternately, setting a value of "true" would send a notification for *every* message,
    while a value of "false" would *never* send a notification.

    For a more complicated example, the value of "client_count_less_than and highlight and
    (last_active or last_notification or replied) and nick_blacklist" would send a
    notification if any of the three conditions in the sub-expression are met, while still
    requiring all of the conditions outside of the parentheses to also be met.

*   `query_conditions = "all"`

    This option is more or less identical to `channel_conditions`, except that it is used
    to filter notifications for private messages.


Roadmap
-------

### Settings

*   Customizable notification titles and message formats.


License
-------

This project is licensed under the MIT license.  See the `LICENSE` file for details.



[mantis]: http://leetcode.net/mantis
[ZNC]: http://en.znc.in "ZNC, an advanced IRC bouncer"
[ISO 8601]: http://en.wikipedia.org/wiki/ISO_8601 "ISO 8601 Date Format"

<!-- vim:set ft= expandtab tabstop=4 shiftwidth=4: -->
