Title: Workaround for newsboat not able to access Tumblr RSS feed
Date: 2018-08-07 12:15
Modified: 2019-03-10 15:15
Category: configs
Tags: newsboat, Tumblr, RSS
Slug: workaround-newsboat-tumblr
Authors: Hendrik Rosendahl
Summary: A configuration workaround for Tumblr's GDPR notice in their RSS feed for newsboat

**Update:** I got mixed results using the previously mentioned `Googlebot` user agent, so I updated the post with a newly working user agent string.

Recently [newsboat](https://newsboat.org/) displayed the following error while retrieving news from a Tumblr RSS feed:
```
Error while retrieving http://hopooo.tumblr.com/rss: unsupported feed format
```
going to that URL in my browser showed a notice about the new GDPR rules and a button to accept those new terms:

![GDPR Notice](../images/01-gdpr_notice.png)

Accepting the notice forwards you to the actual RSS feed.
Further inspection shows that Tumblr sets a cookie, that I accepted the GDPR notice and does not show it in subsequent visits, **but** the RSS reader newsboat does not handle cookies.

A workaround was posted for another RSS reader on their [GitHub page](https://github.com/miniflux/miniflux/issues/140#issuecomment-408366528), there it is noted, that if the user agent contains the word *Googlebot*, then Tumblr does not show the GDPR notice anymore.

For some weeks now this workaround produces mixed results, where most of the time the unsupported feed format error came up again.

So I tried to download the RSS feed with `wget`, where to my surprise no problems occurred.
Then I set the user agent to `Googlebot` by running the command
```bash
$ wget http://hopooo.tumblr.com/rss -U 'Googlebot'
```
and sure enough the resulting page was the GDPR notice of Tumblr.
I also tried to send no user agent string at all, but also then only the GDPR notice was coming back from Tumblr.

So I researched which user agent string `wget` uses when no special string is supplied and it is of the form `Wget/[version]`.

So putting
```
user-agent "Wget/1.20.1"
```
into your newsboat configuration file will make Tumblr RSS feed work again in newsboat… (at least for now).
The configuration file of newsboat is the `$XDG_CONFIG_HOME/newsboat/config` file (which should be `~/.config/newsboat/config` for the most people), or if you do not follow the [XDG basedir standard](https://standards.freedesktop.org/basedir-spec/basedir-spec-latest.html) then the `~/.newsboat/config` file.
