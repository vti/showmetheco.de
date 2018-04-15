Title: Publishr — publish everywhere
Tags: Perl

For <http://pragmaticperl.com> I needed a tool to post the new issue
announcement to several social networks. It ended up supporting Facebook,
Twitter, LiveJournal, VK, Email, IRC, Jabber/XMPP, Skype and more.

[cut]

## Where to get it?

<http://github.com/vti/publishr>.

## How to run

From the git repository:

```
perl -Ilib script/publishr --config publishr.json message.txt
```

Where `message.txt` looks like:

```
Status: This is the short title
Link: http://link-to-the-press-release
Image: /path/to/image.jpg
Tags: perl, pragmaticperl, journal

The
multiline
body
```

Of course every social network supports different kind of messages. This is
handled by the so called channels. For example for Twitter `publishr` only uses
`Status`, `Link` and `Image`.

The `publishr.json` configuration files looks like:

```
{
   "access" : [
      {
         "name" : "twitter access #1",
         "options" : {
            "access_token" : "",
            "access_token_secret" : "",
            "consumer_key" : "",
            "consumer_secret" : ""
         },
         "type" : "twitter"
      }
   ],
   "scenarios" : [
      {
         "access" : "twitter access #1",
         "name" : "post to pragmaticperl twitter",
         "options" : {}
      }
   ]
}
```

`access` is a list of `channel` credentials. You name them as you like, provide
required options and use in `scenarios`. This is made so you can use the same
access tokens for different scenarios, like posting to different Facebook groups
etc.

In `scenarios` you can provide `options` with additional options, like an IRC
channel etc.

## Custom commands

Sometimes you will need to just run a custom cli program. For example this is
how sending to Skype is done. In `util` directory you can find a `skype-chat.py`
Python script which uses [Skyp4Py](https://github.com/awahlig/skype4py) library.
In order to call that script you configure `cmd` scenario:

```
{
    "name":"skype",
    "access":"cmd",
    "options":{
        "env":{
            "PYTHONPATH":"/path/to/skype4py/"
        },
        "cmd":"./util/skype-chat.py 'Skype Chat' '%status% %link%'"
    }
}
```

Where `%status%` and `%link%` are replaced by values from the `message.txt`.

## Running only specific scenarios or channels

Sometimes you would want to run just a specific scenario or a channel, there are
options for this:

```
perl -Ilib script/publishr --config publishr.json \
    --scenario 'post to twitter' message.txt
```

```
perl -Ilib script/publishr --config publishr.json \
    --channel 'facebook' message.txt
```
