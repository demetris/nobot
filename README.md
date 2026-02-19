
nobot is a detection system for low-effort bots that consists of a set of configuration modules for Apache 2.4.

It detects and blocks bots by looking for discrepancies between:

1.  what the agent of the HTTP request claims to be, and
2.  what the HTTP request header says about the agent.

For example, it denies requests that are made by an agent identifying as Chrome 144
but look nothing like the requests Chrome 144 really makes.

nobot also has modules that deny requests by bots that identify themselves honestly but are simply not wanted.
These modules use no detection other than looking at the User-Agent string.
They are the *opinionated* part of nobot.

nobot is fully modular. You can use as little or as much as you want –
anything from two modules to the whole set.

nobot had been tested with Apache versions 2.4.65 and 2.4.66.


Usage
----------------------------------------

Clone the nobot repo somewhere on your server using Git.

```bash
git clone https://github.com/demetris/nobot.git /path/to/your/nobot/clone
```

Add the following to your HTTPS virtual host,
preferably before any other rewrite rules in it:

```apache
<IfDefine !NOBOT_ROOT>
  Define NOBOT_ROOT /path/to/your/nobot/clone
</IfDefine>

##
##  Turn the rewrite engine on if it is not already on.
##
RewriteEngine On

Include ${NOBOT_ROOT}/apache/01-base.conf
Include ${NOBOT_ROOT}/apache/40-unexpected-http-10.conf
Include ${NOBOT_ROOT}/apache/42-unexpected-http-2.conf
Include ${NOBOT_ROOT}/apache/50-unexpected-tls-version.conf
Include ${NOBOT_ROOT}/apache/60-unexpected-header-fields.conf
Include ${NOBOT_ROOT}/apache/61-unexpected-user-agent.conf
Include ${NOBOT_ROOT}/apache/62-unexpected-referer.conf
Include ${NOBOT_ROOT}/apache/63-unexpected-accept.conf
Include ${NOBOT_ROOT}/apache/64-unexpected-encoding.conf
Include ${NOBOT_ROOT}/apache/65-unexpected-language.conf
Include ${NOBOT_ROOT}/apache/66-unexpected-client-hints.conf
Include ${NOBOT_ROOT}/apache/67-unexpected-fetch-metadata.conf
```

Then reload your Apache configuration.

The above set of modules should be safe to include in any Apache configuration,
to the extent that anything in nobot can be said to be safe of course.

If you want to include more modules, look at the next two sections,
and also read the documentation in the file header of each module.

**Do not** just include everything!

For a **minimal** setup, to just see how nobot works with a site of yours, do this:

```apache
<IfDefine !NOBOT_ROOT>
  Define NOBOT_ROOT /path/to/your/nobot/clone
</IfDefine>

##
##  Turn the rewrite engine on if it is not already on.
##
RewriteEngine On

Include ${NOBOT_ROOT}/apache/01-base.conf
Include ${NOBOT_ROOT}/apache/40-unexpected-http-10.conf
Include ${NOBOT_ROOT}/apache/42-unexpected-http-2.conf
```



nobot Apache configuration modules
----------------------------------------

**`O`** stands for *opinionated*  
**`D`** stands for *deterministic*  
**`P`** stands for *probabilistic*  
**`E`** is the client error code or codes the module responds with

| Module                                                                                 | Description/Target                                      | O | D | P | E         |
|:-------------------------------------------------------------------------------------- |:------------------------------------------------------- |:- |:- |:- |:--------- |
| [00-user.conf.example](apache/00-user.conf.example)                                    | Example module for user rules and env vars\*            |   |   |   | N/A       |
| [01-base](apache/01-base.conf)                                                         | Definitions and environment variables                   |   |   |   | N/A       |
| [10-unwanted-bots](apache/10-unwanted-bots.conf)                                       | Honestly named but unwanted bots                        | Y |   |   | 402       |
| [20-unwanted-tools-bannable](apache/20-unwanted-tools-bannable.conf)                   | Unwanted tools and libraries, special edition           | Y |   |   | 402       |
| [25-unwanted-tools-forbidden](apache/25-unwanted-tools-forbidden.conf)                 | Unwanted tools and libraries                            | Y |   |   | 403       |
| [30-anonymous](apache/30-anonymous.conf)                                               | Anonymous requests (empty UA)                           | Y |   |   | 402       |
| [35-anonymous-essentially](apache/35-anonymous-essentially.conf)                       | Essentially anonymous requests                          | Y |   |   | 402       |
| [40-unexpected-http-10](apache/40-unexpected-http-10.conf)                             | Unexpected HTTP/1.0                                     |   | Y |   | 451       |
| [41-unexpected-http-11](apache/41-unexpected-http-11.conf)                             | Unexpected HTTP/1.1 + more signals†                     |   | Y |   | 451       |
| [42-unexpected-http-2](apache/42-unexpected-http-2.conf)                               | Unexpected HTTP/2                                       |   | Y |   | 451       |
| [45-unexpected-http-11-for-non-wp](apache/45-unexpected-http-11-for-non-wp.conf)       | Unexpected HTTP/1.1 for a non-WordPress site†           |   | Y |   | 451       |
| [50-unexpected-tls-version](apache/50-unexpected-tls-version.conf)                     | Unexpected TLS version                                  |   | Y |   | 451       |
| [60-unexpected-header-fields](apache/60-unexpected-header-fields.conf)                 | Unexpected header fields                                |   | Y |   | 451       |
| [61-unexpected-user-agent](apache/61-unexpected-user-agent.conf)                       | Unexpected UA (made-up or malformed)                    |   | Y |   | 451       |
| [62-unexpected-referer](apache/62-unexpected-referer.conf)                             | Unexpected Referer                                      |   | Y |   | 451       |
| [63-unexpected-accept](apache/63-unexpected-accept.conf)                               | Unexpected Accept                                       |   | Y |   | 451       |
| [64-unexpected-encoding](apache/64-unexpected-encoding.conf)                           | Unexpected Accept-Encoding + more signals               |   | Y |   | 451       |
| [65-unexpected-language](apache/65-unexpected-language.conf)                           | Unexpected Accept-Language + more signals               |   | Y |   | 451       |
| [66-unexpected-client-hints](apache/66-unexpected-client-hints.conf)                   | Unexpected Sec-CH-\* fields                             |   | Y |   | 451       |
| [67-unexpected-fetch-metadata](apache/67-unexpected-fetch-metadata.conf)               | Unexpected Sec-Fetch-\* fields                          |   | Y |   | 451       |
| [70-unexpected-other](apache/70-unexpected-other.conf)                                 |                                                         |   | Y |   | 451       |
| [80-bot-impersonators](apache/80-bot-impersonators.conf)                               | Bots that impersonate other bots‡                       |   | Y |   | 451       |
| [98-outdated-browsers](apache/98-outdated-browsers.conf)                               | Old browsers that bots like to impersonate              | Y |   | Y | 426       |

NOTES

`*` Can be used to whitelist requests. See `NOBOT_ALLOWED_BY_USER` section below.  
`†` Module depends on server configuration. See section below.  
`‡` Module may start giving false positives if left unmaintained. See section below.



Modules that need special care (do NOT just include those)
----------------------------------------

### Opinionated modules

As the name says, these are the opinionated nobot modules.
Do **not** include opinionated modules unless you have reviewed them and know what they block.

### Configuration-dependent modules

These are modules that should only be included in specific configurations.

They are two:

-   [41-unexpected-http-11](apache/41-unexpected-http-11.conf)
-   [45-unexpected-http-11-for-non-wp](apache/45-unexpected-http-11-for-non-wp.conf)

Only use them if HTTP/2 is enabled on the server!

Only use the second if the site is not WordPress.

### bot-impersonators module

Most detection in nobot will work well indefinitely if it works well now:

A Safari 16 that sends *gzip, deflate, br, zstd* as Accept-Encoding will be as wrong in five years as it is now.

A Firefox 12 that connects with HTTP/2 will be as unbelievable in five years as it is now.

A Chrome 60 that connects with TLS 1.3 is a perfect impossibility.

However, some nobot rules may break if external things change.
This is true for the [rules that block bot impersonators based on whitelists](apache/80-bot-impersonators.conf).
If, for example, Google starts using a range that is not whitelisted in nobot, its bots will get blocked by the rules in the 80 module.

Be aware of that.

### outdated-browsers module

The purpose of [the outdated-browsers module](apache/98-outdated-browsers.conf) is to catch evading bots, not just block old browsers,
but it does block a lot of old browsers, and it can claim innocent victims (real humans using those browsers).

Only include the outdated-browsers module if you understand what it does and if you agree with its approach.

If you include it, you **should** add a custom error document for 426 that explains how to bypass the block (refresh the page).

The easiest way to add a custom message is maybe this, in your main Apache configuration:

```apache
ErrorDocument 426 "Please update your browser. <br />If you cannot or prefer not to, you can refresh the page to continue."
```


`NOBOT_ALLOWED_BY_USER`
----------------------------------------

You can whitelist requests before they get to nobot rules by setting the `NOBOT_ALLOWED_BY_USER` environment variable.

[00-user.conf.example](apache/00-user.conf.example) has a few examples.

Whitelisted requests skip all nobot rules. They cannot be blocked by nobot.

Whitelisting is useful for two cases:

1.  For requests that are blocked by a nobot module you use
2.  For requests that you trust and that it doesn’t make sense wasting time to check



Debugging nobot
----------------------------------------

For debugging, or if you are just curious to see what module each 402/403/451 comes from,
you can use a custom log format.

This is the full log format I use for logging everything I want to know about:

```apache
LogFormat "%h %l %u %t \"%r\" %>s %O \"%{Referer}i\" \"%{User-Agent}i\" \"%{Accept-Encoding}i\" \"%{Accept-Language}i\" \"%{Sec-CH-UA}i\" \"%{Sec-CH-UA-Mobile}i\" \"%{Sec-CH-UA-Platform}i\" \"%{Sec-Fetch-Dest}i\" \"%{Sec-Fetch-Mode}i\" \"%{Sec-Fetch-Site}i\" \"%{Sec-Fetch-User}i\" \"%{Sec-Purpose}i\" \"%{Priority}i\" \"%{Accept}i\" \"%{X-Forwarded-For}i\" \"%{X-Forwarded-Proto}i\" \"%{X-Real-IP}i\" \"%{Forwarded}i\" \"%{Via}i\" \"%{CF-Worker}i\" \"%{CF-Connecting-IP}i\" \"%{SSL_PROTOCOL}x\" \"%{NOBOT}e\"" combined_extra_headers
```



Client error codes used in nobot
----------------------------------------

nobot (ab)uses three client error codes for the purpose of tagging the requests it denies.

You can use the three codes to automatically ban the addresses of the requests,
or to filter out the bad bots in your server-side analytics.
Or for anything else you might think of.

The meaning of the three client error codes is re-interpreted in the following way:

### 402

If you really wanted access to this specific website, you would have to pay.

### 426

Please update your browser.

*If you use the outdated-browsers module (the only one in nobot that responds with 426),
make sure to include a custom error message or document for 426.
See above for an example.*

### 451

Lying is illegal on this site.



Bad bots nobot can and cannot stop, and how to stop more bad bots
----------------------------------------

By its nature–stateless detection based on what the web server can know in the context of a single HTTP request–the nobot system cannot stop all bad bots.

The bots that evade detection in nobot fall into two main categories:

**A**. Well-made bad bots that can’t run JavaScript. For those you can use a system like [Anubis](https://anubis.techaro.lol/). It stops all of them.

**B**. Well-made bad bots that can run JavaScript. Those use real browser engines rather than HTTP client libraries disguised as browsers.

For purposes of blocking, the **B** bots fall into two subcategories:

**B1**. Well-made bad bots from ASNs that send only bad traffic. Such ASNs are safe to block at the firewall level.
[ipverse/as-ip-blocks](https://github.com/ipverse/as-ip-blocks) is a well-maintained project that helps with that.

After that point it gets difficult, as the complexity and cost increase:

**B2**. Well-made bad bots that come from ASNs that send mixed traffic.

If it is a datacenter ASN, you can block the whole thing after whitelisting the ranges of good services and bots you need from it.

If it is an ISP ASN (residential proxies), blocking the whole ASN after whitelisting is not an option.
For those cases you need to do expensive work or/and pay for expensive services.



robots.txt
----------------------------------------

Relying on robots.txt has become problematic. It entails an unsustainable cognitive load.
You cannot possibly know or aspire to know which ones of all the bots respect and which ones do not respect robots.txt.

For my personal sites I just block the bots I don’t want (the opinionated modules in nobot)
and use robots.txt only for bots that are known to respect it.
[My robots.txt template](robots-txt/robots.txt).




Resources
----------------------------------------

Resources that have been especially useful for working on nobot:

-   [Apache HTTP Server Documentation](https://httpd.apache.org/docs/)
-   [Can I use...](https://caniuse.com/)
-   [go-httpbin(1): HTTP Client Testing Service](https://httpbingo.org/)
-   [IPinfo](https://ipinfo.io/)
-   [MDN Web Docs](https://developer.mozilla.org/)
-   [zytrax.info - Browser ID (User-Agent) strings](https://www.zytrax.com/tech/web/browser_ids.htm) (for old browsers that are not easy to test on real hardware)
-   [zytrax.info - Mobile browser ID (User-Agent) strings](https://www.zytrax.com/tech/web/mobile_ids.html) (for old *mobile* browsers that are not easy to test on real hardware)



Development
----------------------------------------

After cloning the repo, please enable the pre-commit hook:

```
git config core.hooksPath scripts/hooks
```



Why nobot
----------------------------------------

nobot started as an experiment and is still one to a large extent:

I wanted to see if I can tag and block a sufficient amount of annoying and lying bots
based solely on what the web server knows in the context of a single HTTP request.

This type of server-side detection has two main advantages:

1.  It is transparent to the user (no front-end challenges)
2.  It works without an extra application in the stack

The question is: Does it work well enough?



License
----------------------------------------

nobot is original work and is published under the CC BY-SA 4.0 license.
See [LICENSE.txt](LICENSE.txt) and [LEGALCODE.txt](LEGALCODE.txt).
