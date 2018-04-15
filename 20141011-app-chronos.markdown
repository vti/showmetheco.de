Title: App::chronos - automatically record your computer activities
Tags: Perl, productivity

Often you want to know how much time you spend on a random computer activity
during your work/home day. There are lots of apps that allow you to record the
time, unfortunately you have to manualy turn them on and off. It can be really
frustraiting when you forget to do so. So I have written an app that does that
automatically.

[cut]

[chronos](http://github.com/vti/app-chronos) listens for X11 window switches and
records how much time you have spent on every application. It runs a set of
filters that guess the type of the application and its name. Moreover if the
application can answer more than a question "what type am I" and additionally
can provide like a visited URL, a contact you're chatting with and so on, then
the filter can parse that information and add to the log.

## Activity details

As previously said the filters can parse additional information. For example
right now if the application is a Firefox or a Chromium browser than the
currently visited URL is detected. This is done by parsing current sessions. In
case of a Skype or Pidgin, for example, the current contact name is detected.

## Output

`chronos` prints the events to the stdout, so the log can be easily saved to
any file you like. The format is simple: a single line in JSON, with UNIX epoch
timestamps. For example:

```
{
   "_end" : 1412750698,
   "_start" : 1412750693,
   "application" : "Chromium",
   "category" : "browser",
   "class" : "\"Chromium\", \"Chromium\"",
   "command" : "",
   "id" : "0x4a00048",
   "name" : "\"reddit: the front page of the internet - Chromium\"",
   "role" : "\"browser\"",
   "url" : "www.reddit.com"
}
```

The JSON part has several fields. `role`, `class`, `name` and `command` are the
fields recorded from X11 and they are saved as is. The filter program could for
example detect what kind of a command line I am running (this time vim) and
what kind of a file I am working on.

## Reporting

Reading and analyzing the log file isn't very handy, that is where the `report`
command steps in.

As you already know the event is a JSON object that has various fields. The
`report` tool can search through those events, group the results and sort the
results by the time spent on them.

### Show top 10 visited URLs:

```
$ chronos report --fields 'url' --where '$category eq "browser"' --group_by 'url' log_file | head -n 10

00d 00:18:27 url=www.youtube.com
00d 00:05:29 url=github.com
00d 00:01:59 url=twitter.com
00d 00:01:25 url=code.google.com
```

Here I am showing only `url` field, searching for `category` named `browser`
and group by `url`.

Using `--where` and `--group_by` various useful reports can be produced specific
to your needs.

### `--where` syntax

If you have noticed option `--where` has a Perl-like syntax. That is actually
`eval`-ed into a Perl subroutine that is than run on every event. This way the
`where` clause can be as profound as needed.

### `--from` and `--to`

## Timeout

To configure how often you want `chronos` to sleep before recording any activity
use `--timeout` option.

## Idle time

`chronos` also detects the idle time and stops recording the activity. Idle time
is detected by running `xprintidle` and comparing it to the `--idle_timeout`
option, which is 5 minutes by default. So if you don't type anything or don't
move your mouse for 5 minutes the previous activity is considered as ended.

## Flushing

Various bad things can happen during recording. This could be the power outage
or accidental killing of the `chronos` process. In order to be more robust
`chronos` periodically flushes the activity to the log file. This can be
configured by `--flush_timeout` option. And you won't loose the event recording
when you've been working on it for several hours.

## Contributing

Different people use different applications. I cannot write the filters for
every application out there, so if you use `chronos` and want an application and
its options to be parsed, just write a filter package, it's as simple as:

```
package App::Chronos::Application::Skype;

use strict;
use warnings;

use base 'App::Chronos::Application::Base';

sub run {
    my $self = shift;
    my ($info) = @_;

    # It's not a Skype application
    return
      unless $info->{role} =~ m/ConversationsWindow/
      && $info->{class} =~ m/Skype/
      && $info->{name} =~ m/Skype/;

    # Yay, it's Skype, let's parse the contact name
    $info->{application} = 'Skype';
    $info->{category} = 'im';
    ($info->{contact}) = $info->{name} =~ m/^"(?:\[\d+\])?(.*?) - Skype/;

    return 1;
}

1;
```

## Tips & tricks

I personally have a bash script that combines several reports:

```
#!/bin/sh

LOG_FILE=$1
LIMIT=10
COMMAND="perl -Ilib script/chronos"

echo 'Top categories:'
$COMMAND report --fields 'category' --group_by 'category' $LOG_FILE
echo
echo "Top $LIMIT talks:"
$COMMAND report --fields 'contact' --where '$category eq "im"' --group_by 'contact' $LOG_FILE | head -n $LIMIT
echo
echo "Top $LIMIT URLs:"
$COMMAND report --fields 'url' --where '$category eq "browser"' --group_by 'url' $LOG_FILE | head -n $LIMIT
echo
echo 'Idle time:'
$COMMAND report --where '$idle' $LOG_FILE
```

And then:

```
$ ./report.sh log_file | mail -s Activities vti
```
