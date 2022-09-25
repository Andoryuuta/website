---
title: ".edu domains are not credible"
date: 2022-09-07T22:02:30-04:00
draft: false
---

It's quite difficult to find real, credible, information on the internet nowadays.

Something that has made this easier (for me, at least) has been to search specifically for content on `.edu` domains, since that TLD is restricted to accredited institutions. This is as simple as prefixing your Google search with `inurl:edu `.

I've been doing this for a while with no issues, so you can imagine my surprise when I recently tried to find information about some "fat burner" pills that were being sold on Amazon and searched for `inurl:edu nobi fat burner` in the off-chance it was mentioned in some study/paper.

![Screenshot of google search results showing .edu websites with advert titles, such as "5 Best Fat Burners for Women 2022"](first_search_results.png)

Dang! If UMass Med, Penn State, _and_ UC Santa Cruz are recommending these things, then they must be safe and effective.

However, just to do my due diligence, I should probably at least click on the first link and see what it says about-

Oh, that's odd.

This doesn't look like a university website?

![Screenshot of page redirected to by .edu site. The page shows the "Fox News" logo and contains an article about a "No-Excercise Skinny Pill". The page is hosted on afnethealthy.com](redirected_first_page.png)

Maybe I clicked the wrong one, let me go back and copy and paste the URL.

Huh, a 404 page this time?
![Screenshot of the 404 "Not Found" page on the real UMass Med website](umassmed_real_site_404.png)
Weird. Oh well.

If three universities  are recommending these entirely unregulated pills to me, then they must be good!

<br>
... Alright, I'll drop the schtick.

<br>
<br>

# What's happening here?

I decided to look a little bit deeper into the first search result.

First, I took the raw URL and `curl`ed it, but I got a 404. For some reason, it didn't redirect to the spam site.
![Screenshot `curl` output of the URL. The site returned a 404 "Not Found"](normal_curl.png)

My first instinct here was that _something_ was checking if the client was the "expected" browser somehow. For example, it could be checking if the `User-Agent`, `Referer`, or `Accept` headers matches what would be sent from Chrome, [TLS fingerprinting](https://daniel.haxx.se/blog/2022/09/02/curls-tls-fingerprint/), etc.

Thankfully it was one of my first guesses. Adding the `Referer` of `https://www.google.com/` did the trick:
![Screenshot `curl` output of the URL with the google referer header. The site returned a 302 "Found" redirect to the spam site](curl_referer_difference.png)

Admittedly, at this point I was a _tiny_ bit worried that this was something on my local machine, so I opted for the most optimal solution for checking: Sending my brother an unprompted discord message asking him to `curl` a suspicious URL with no context. Sadly it turned out that he _did_, in fact, want context.

How unfortunate.

![Discord chat between my brother and I. Ando(me):"could you curl this for me?", slimsag(brother):"call?"](discord_request.png)

Regardless, I eventually convinced him to run it and confirmed it wasn't an issue on my end.

Now at this point, I knew that it was happening on their server(s), and under which conditions it happened (coming from Google Search results), but I had no way of figuring out _why_ it was happening. (e.g. something in their reverse proxy, some PHP code, etc)

The only indicator I had here was the very unusual `haircki` cookie that was being set when visiting the regular page. It _could_ technically be legitimate, but it seemed extremely out-of-place for a uni website.

By sheer luck, a [sourcegraph search](https://sourcegraph.com/search?q=context:global+haircki&patternType=standard) turned up a result for `haircki`, which lead me to a GitHub repo containing WordPress malware samples:
![sourcegraph.com search results for "haircki", showing multiple GitHub repositories containing the string](sourcegraph_haircki_search.png)

https://github.com/stefanpejcic/wordpress-malware/blob/master/01.06.2021/index.php

After briefly looking over that sample from another site, I feel confident in saying that it produces the same behavior of search engine link hijacking that I'm seeing on the `umassmed.edu` site.

The answer here turned out to be less interesting than I thought, just another case of websites running vulnerable versions of WordPress and/or WP plugins.

# Resolution(?)

I thought initially that I would drop 