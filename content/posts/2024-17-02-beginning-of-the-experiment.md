---
title: "Beginning of the experiment"
date: 2024-17-02T09:49:03+10:00
author:
  name: "Yevhenii Kurtov"
  image: images/author/sage-kirk.jpg
  twitter: '@yevkurtov'
categories: ["Agile", "Lean", "Startup"]
tags: ["Entrepreneurship", "DevOps"]
description: Discovering the idea, settling into a mindset and building the necessary pieces to get going.
thumbnail: "https://errencal.sirv.com/grocery-times/first-post-image.png?w=640&h=360"
image: "https://errencal.sirv.com/grocery-times/first-post-image.png"
---

I got the idea for Grocery Times a day after following https://traderjoesprices.com/ on Hacker News. It felt joyful and had no clear monetisation path, just pure fun.
I even joked about it with colleagues. We had a couple of laughs discussing betting on eggs & milk prices and what derivative markets can come from it.
At that moment, the idea paid for itself with good emotions, and I decided to see how much more I could squeeze out of it.

## Mindset

Currently, I approach building a startup as an experiment -  there is a hypothesis to be validated, and I will learn something along the road.
Grocery Times is a perfect training ground because it's fun, and I don't expect it to generate any direct monetary income (which doesn't mean that I don't hope it to).
Operating from that place gives me clarity on where I am - if it's fun, then everything is going in the right direction. If it's not - then something is wrong.
__"Is this fun?" is a straightforward indicator to weigh any decision against__. "Is it fun to build that infra? Hell yes!" and then I observe my state. If the next step doesn't excite me, I look for another way forward.

## Starting off

So, the product is simple - it will track grocery price movements in UK supermarkets.
My nearest supermarket is Sainsbury's, and that's where I  started.

> Idea: I'm a man of simple habits - I mostly buy the same products. Can this product track a predefined basket of products for me so that I will see how its cost is moving on?

Obviously, because I knew I would scrape their website, I first checked what I had in the toolbox and found - [Crawly](https://github.com/elixir-crawly/crawly). Gotta scratch a builder syndrome itch first.

Next, I went on their website and took a deep dive on an emotional rollercoaster -  it turned out to be dynamically rendered and required a JavaScript-capable crawler.
I looked into options offered by Crawly and discovered that it provides an [out-of-the-box integration with Splash](https://hexdocs.pm/crawly/Crawly.Fetchers.Splash.html) - a lightweight javascript rendering service explicitly developed for scraping by Zyte (formerly ScrapingHub). Which somehow didn't feel fun at all. It could be because it mentioned Lua for scripting, and I didn't feel like learning a new programming language was part of the main quest.


Sure enough, Sainsbury's website has an anti-scraping protection that required setting headers, and I also had to instruct the browser to wait until a specific part of the page loads. After spending 20 minutes hacking something together and observing how I didn't like the interaction model, I decided to find another way.

_Note: they have a perfectly adaptable  ['wait-for-element' example](ttps://github.com/scrapinghub/splash/blob/master/splash/examples/wait-for-element.lua). example_

### Roll your own

A short research with Google showed that Playwright could be what I'm looking for. It took me exactly ten minutes to throw together PoC and ensure that I was on the right track

I extracted that piece in a standalone Express API application and tested it wis CURL. It worked just right!

![playwright poc](https://errencal.sirv.com/grocery-times/playwright-poc-screenshot.png)

```js
const { test, expect } = require("@playwright/test");

test("renders ui", async ({ page }) => {
  await page.goto(
    "https://www.sainsburys.co.uk/gol-ui/groceries/dietary-and-world-foods/world-foods/c:1034136?tag:c006:position1:Shop-world-foods:nonvalue",
  );
  await expect(page.locator("section#main")).toBeVisible();
});
```

I extracted that piece in a standalone Express API application and tested it with CURL. It worked just right!

![scraper-api](https://errencal.sirv.com/grocery-times/scraper-api-screenshot.png)

## What about the data?

With the good enough scraping capability to take me off the ground, I began investigating how I move around the website and where I get product information. Needless to say, Sainsbury's web store is precisely at the level of quality one would expect from the digital product created by a large UK company established in the XIX century. Sometimes integration is a pain, but that's precisely why I'm not particularly worried about their anti-scraping measures.

Below is the excerpt of my message at the #london in the Elixir slack.

1. Some categories don't have a "view all" page to browse the section's catalogue. For example, [Dietary & World Foods](https://www.sainsburys.co.uk/gol-ui/groceries/dietary-and-world-foods/c:1019265) have "View all" (see screenshot 1), but [Fruit & Vegetable](https://www.sainsburys.co.uk/gol-ui/groceries/fruit-and-vegetables/c:1020082) does not.

![Screenshot 1](https://errencal.sirv.com/grocery-times/screenshot-1-sainsburys-day-1.png)

  Instead, it's possible to navigate into subsections by following the links at the top of the page (see screenshot 2).

![Screenshot 2](https://errencal.sirv.com/grocery-times/screenshot-2-sainsburys-day-1-up-2.png)

  I noticed at least four taxonomy levels, maybe even more, and significantly enough subcategories might not be linked in the parent's header section. For example, "Milk" is not listed on the "Dairy, eggs & chilled" header section (see screenshot 3).

  ![Screenshot 2](https://errencal.sirv.com/grocery-times/screenshot-3-sainsburys-day-1.png)

  That led me to the conclusion that navigating the menu is the most reliable way to get the items. That seems to be a lot of work and potentially a maintenance burden, as the menu might change in many different ways. Which made me think that if they want their catalogue to be available through search, they may expose it through a sitemap.


 2. I downloaded their sitemap and checked for the product from the "Milk" section. It was listed under four different marketing categories in the biggest XML file. Unfortunately, it was not listed under its direct link https://www.sainsburys.co.uk/gol-ui/product/sainsburys-british-semi-skimmed-milk-113l-2-pint-, which suggests a bit more work for a parser.

 ```bash
 grep 'sainsburys-british-semi-skimmed-milk-227l-4-pint-' *
 wcsstore-robots-sitemap_10151_1.xml.gz:    <loc>https://www.sainsburys.co.uk/shop/gb/groceries/product/details/pancakeday/sainsburys-british-semi-skimmed-milk-227l-4-pint-</loc>
 wcsstore-robots-sitemap_10151_1.xml.gz:    <loc>https://www.sainsburys.co.uk/shop/gb/groceries/product/details/aldi-price-match/sainsburys-british-semi-skimmed-milk-227l-4-pint-</loc>
 wcsstore-robots-sitemap_10151_1.xml.gz:    <loc>https://www.sainsburys.co.uk/shop/gb/groceries/product/details/first-shop-essentials/sainsburys-british-semi-skimmed-milk-227l-4-pint-</loc>
 wcsstore-robots-sitemap_10151_1.xml.gz:    <loc>https://www.sainsburys.co.uk/shop/gb/groceries/product/details/nectar--/sainsburys-british-semi-skimmed-milk-227l-4-pint-</loc>
 wcsstore-robots-sitemap_10151_1.xml.gz:    <loc>https://www.sainsburys.co.uk/shop/gb/groceries/product/details/make-a-smoothie/sainsburys-british-semi-skimmed-milk-227l-4-pint-</loc>
 wcsstore-robots-sitemap_10151_1.xml.gz:    <loc>https://www.sainsburys.co.uk/shop/gb/groceries/product/details/dairy-and-chilled-essentials/sainsburys-british-semi-skimmed-milk-227l-4-pint-</loc>
 ```

 ### Downloading sitemaps

As a quick note, I want to say that using Nushell to put together a script to download sitemaps was really fun!

And having all of them locally allowed me to spot that the same product is featured multiple times. The biggest sitemap was 10 MB large with 50k entries. Fortunately, I was on a real roll with Nushell, and it was a pleasure to put together another script that filtered unique entries by products, which conveniently were the last segments of the URL.


 ```nu
 #!/usr/bin/env nu

 def curlh [url: string] { curl -H 'User-Agent: Mozilla/5.0 (Macintosh; U; Intel Mac OS X 10_2_9) AppleWebKit/603.35 (KHTML, like Gecko) Chrome/48.0.3351.177 Safari/603' -H 'Accept-Language: en-GB,en;q=0.5' $url };

 let domains = ["https://www.sainsburys.co.uk/"]
 let $tmp_dir = "downloaded"

 $domains | each { |domain|
   curlh ($domain + "sitemap.xml") |
   grep loc |
   lines |
   str trim |
   parse '<loc>{url}</loc>' |
   get url |
   str trim |
   each {|url|
     let download_dir =  ($tmp_dir + "/" + ($domain | url parse  | get host) + "/")
     mkdir $download_dir
     let filename = ($url  | str replace $domain "" | str replace -a "/" "-" );
     curlh $url | save -f ($download_dir + $filename)
   }

   echo $"🏪 (ansi rb) ($domain)(ansi green)  sitemap saved(ansi reset)"

 }
```

And having all of them locally allowed me to spot that the same product is featured multiple times. The most extensive sitemap was 10 MB large with 50k entries. Fortunately, I was on a roll with Nushell, and it was a pleasure to put together another script that filtered unique entries by products, which conveniently were the last segments of the URL.

```nu
#!/usr/bin/env nu

def main [filepath: string] {
  let output_filename = ("products-" + ($filepath | path basename ) + ".txt");
  grep -h '<loc>' $filepath |
  lines |
  str trim |
  parse '<loc>{url}</loc>' |
  get url  |
  each { |it|
    let product = ($it | path basename);
    $it |
    each {
      { product: $product, url: $it }
    }
  } |
  uniq-by product |
  get url |
  where { |it| $it | str starts-with "https://" } | # Sainsbury's has broken links in their sitemaps
  save -f $output_filename;

  echo $"(wc -l $output_filename | into string | str trim | split row ' ' | first) unique products saved in ($output_filename)";
}
```

The algorithm builds a list of maps with `url` and `product`, where `product`'s value is the last `url`'s segment. Then, it filters out that list by the value of `product`. Putting everything in one map containing products as keys and URLs as values would be more efficient, but I couldn't be bothered how to figure it out as it was already too late. I knew I could do it later in the Elixir, and that felt safe enough for me.

```bash
17699 unique products saved in products-wcsstore-robots-sitemap_10151_1.xml.gz.txt

________________________________________________________
Executed in   27.75 secs    fish           external
   usr time   27.61 secs   74.00 micros   27.61 secs
   sys time    0.06 secs  756.00 micros    0.06 secs
```

Back of the napkin calculation suggested that bringing the number of links in the most extensive files from 50k to 17.7k would make it possible to crawl all of them with RaspberyPI, on which I planned to host the Scraper API.


![RaspberryPI 4](https://errencal.sirv.com/grocery-times/btm-load-rpi-10-req.png)

Running four requests per core on four cores with 20 seconds each to parse 24399 URLs from all sitemaps `(24399  / 16) / 3 / 60 =~ 8.47 hrs`.
That leaves enough spare capacity.

## PoC Infrastructure

At this moment, I was heavily invested in making Scraper API up and running under the condition that the weather may change, and in the future, I may want to start building its ant-scraping capabilities on short notice.
That aligned with the spirit of the experiment with which I started the project. I didn't have any grand plans for it, _I didn't expect it to work at all_. I even considered the first run against the live data as a small experiment within an experiment. I knew that road bumps would be imminent, and I would have to intervene in the process multiple times before I could scrape Sainsbury's grocery data.

![network setup](https://errencal.sirv.com/grocery-times/raspberry-pi-setup.svg)

Inspired by ideas from Kamal, I decided to spin up a few containers per core on RaspberryPI and load-balance them with Traefik.

However, halfway down the road, it didn't feel right, and I decided to opt out of managing the process with PM2 instead.

I updated the express startup routine to bind to port 3000 plus the instance number passed through the `NODE_APP_INSTANCE` environment variable by PM2.

```javascript
const express = require("express");
const fetchPageContent = require("./fetchPageContent");

const app = express();
const PORT = 3000 + (parseInt(process.env.NODE_APP_INSTANCE) || 0);
app.use(require("express-status-monitor")());
app.use(express.json());

app.post("/fetch", async (req, res) => {
  const { url, waitFor } = req.body;
  if (!url) {
    return res.status(400).send({ error: "URL is required" });
  }

  try {
    const content = await fetchPageContent(url, waitFor);
    res.header("Content-Type", "text/plain");
    res.send(content);
  } catch (error) {
    console.error(error);
    res.status(500).send({ error: error.message || "Failed to fetch URL" });
  }
});

app.listen(PORT, "localhost", () => {
  console.log(`Server running on http://localhost:${PORT}`);
});

```

For the first deployment, I rsync'ed the whole folder to the RaspberryPI, but it felt cumbersome, and I made an extra step to generate a deployment key, add it to GitHub and be able to pull the code from it on the dev board.

One of my late-night decisions was to generate a standalone SSH key to access the Scraper API project and check it in the repo(encrypted with ansible-vault). Please don't ask me why :)

```
m2 status
┌────┬───────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id │ name      │ namespace   │ version │ mode    │ pid      │ uptime │ ↺    │ status    │ cpu      │ mem      │ user     │ watching │
├────┼───────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 0  │ server    │ default     │ 1.0.0   │ cluster │ 369555   │ 2D     │ 0    │ online    │ 0%       │ 79.2mb   │ ykt      │ disabled │
│ 1  │ server    │ default     │ 1.0.0   │ cluster │ 369565   │ 2D     │ 0    │ online    │ 0%       │ 79.4mb   │ ykt      │ disabled │
│ 2  │ server    │ default     │ 1.0.0   │ cluster │ 1619898  │ 6s     │ 1    │ online    │ 0%       │ 78.4mb   │ ykt      │ disabled │
│ 3  │ server    │ default     │ 1.0.0   │ cluster │ 369590   │ 2D     │ 0    │ online    │ 0%       │ 77.4mb   │ ykt      │ disabled │
│ 4  │ server    │ default     │ 1.0.0   │ cluster │ 369614   │ 2D     │ 0    │ online    │ 0%       │ 78.7mb   │ ykt      │ disabled │
│ 5  │ server    │ default     │ 1.0.0   │ cluster │ 369626   │ 2D     │ 0    │ online    │ 0%       │ 75.6mb   │ ykt      │ disabled │
│ 6  │ server    │ default     │ 1.0.0   │ cluster │ 369638   │ 2D     │ 0    │ online    │ 0%       │ 77.3mb   │ ykt      │ disabled │
│ 7  │ server    │ default     │ 1.0.0   │ cluster │ 369652   │ 2D     │ 0    │ online    │ 0%       │ 79.3mb   │ ykt      │ disabled │
│ 8  │ server    │ default     │ 1.0.0   │ cluster │ 369674   │ 2D     │ 0    │ online    │ 0%       │ 86.2mb   │ ykt      │ disabled │
│ 9  │ server    │ default     │ 1.0.0   │ cluster │ 369708   │ 2D     │ 0    │ online    │ 0%       │ 77.5mb   │ ykt      │ disabled │
│ 10 │ server    │ default     │ 1.0.0   │ cluster │ 369723   │ 2D     │ 0    │ online    │ 0%       │ 79.2mb   │ ykt      │ disabled │
│ 11 │ server    │ default     │ 1.0.0   │ cluster │ 369736   │ 2D     │ 0    │ online    │ 0%       │ 77.4mb   │ ykt      │ disabled │
│ 12 │ server    │ default     │ 1.0.0   │ cluster │ 369768   │ 2D     │ 0    │ online    │ 0%       │ 77.7mb   │ ykt      │ disabled │
│ 13 │ server    │ default     │ 1.0.0   │ cluster │ 369791   │ 2D     │ 0    │ online    │ 0%       │ 77.8mb   │ ykt      │ disabled │
│ 14 │ server    │ default     │ 1.0.0   │ cluster │ 369806   │ 2D     │ 0    │ online    │ 0%       │ 77.6mb   │ ykt      │ disabled │
│ 15 │ server    │ default     │ 1.0.0   │ cluster │ 369819   │ 2D     │ 0    │ online    │ 0%       │ 78.1mb   │ ykt      │ disabled │
└────┴───────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
```

After everything was up and running, I put 2/3 of the setup in the Ansible script and decided to call it a day even though the port number wasn't in the environment variable :)

```yml
# traefik.yml.j2
api:
  dashboard: true
  insecure: true

entryPoints:
  scraper-api-entrypoint:
    address: ":9050"
providers:
  file:
    filename: {{ traefik_dir }}/conf/traefik_dynamic.yml

log:
  level: INFO
```

```yml
# traefik_dynamic.yml.j2
http:
  services:
    scraper-api:
      loadBalancer:
        servers:
{% for port in range(3000, 3000 + ansible_processor_vcpus * 4) %}
          - url: "http://localhost:{{ port }}"
{% endfor %}
  routers:
    scraper-api-router:
      entryPoints:
        - "scraper-api-entrypoint"
      rule: "PathPrefix(`/`)"
      service: "scraper-api"

```
