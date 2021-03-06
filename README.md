# Facebook Page Crawler

Extract public information from Facebook Pages.

## Usage

If you want to run the actor on the Apify platform, you need to have at least some Apify proxy group so that Facebook doesn't block you. Since it uses Puppeteer, the minimum memory for running is 2048 MB.

## Input

Example input, only `startUrls` and `proxyConfiguration` are required (check `INPUT_SCHEMA.json` for settings):

```js
{
    "startUrls": [
        { "url": "https://www.facebook.com/biz/hotel-supply-service/?place_id=103095856397524" }
    ],
    "maxPosts": 3,
    "maxPostDate": "2020-10-10",
    "maxPostComments": 15,
    "maxReviewDate": "10 days", // if today is 2020-04-11, reviews will be 2020-04-01 and beyond
    "maxCommentDate": "1 month",
    "maxReviews": 3,
    "commentsMode": "RANKED_THREADED",
    "scrapeAbout": true,
    "scrapeReviews": true,
    "scrapePosts": true,
    "scrapeServices": true,
    "language": "cs-CZ",
    "proxyConfiguration": {
        "useApifyProxy": true
    }
}
```

## Output

```json
{
  "categories": [
    "Hotel",
    "Lázně"
  ],
  "info": [
    "Luxury 5 star hotel in former monastery complex in Prague, Czech Republic."
  ],
  "likes": 4065,
  "messenger": "https://m.me/1" ...,
  "posts": [
    {
      "postDate": "2020-03-08T15:35:51.000Z",
      "postText": "Our guest " ...,
      "postImages": [
        {
          "link": "https://www.facebook.com/.../photos" ...,
          "image": "https://scontent-prg1-1.xx.fbcdn.net/v/t1.0-0/" ...
        }
      ],
      "postLinks": [],
      "postUrl": "https://www.facebook.com/permalink.php?story_fbid="...,
      "postStats": {
        "comments": 4,
        "reactions": 66,
        "shares": 2
      },
      "postComments": {
        "count": 2,
        "mode": "RANKED_THREADED",
        "comments": [
          {
            "date": "2020-03-08T22:13:10.000Z",
            "name": "Caro" ...,
            "profileUrl": null,
            "text": "Wow..." ...,
            "url": "https://www.facebook.com/.../posts/" ...
          },
          {
            "date": "2020-03-08T16:10:43.000Z",
            "name": "Bri" ...,
            "profileUrl": "https://www.facebook.com/b" ...,
            "text": "Dan" ...,
            "url": "https://www.facebook.com/.../posts/" ...
          }
        ]
      }
    }
  ],
  "priceRange": "$$$$",
  "reviews": {
    "reviews": [
      {
        "title": "Phi" ...,
        "text": "Très "...,
        "attributes": [
          "Romantická atmosféra",
          "Luxusní hotelová kosmetika",
          "Důmyslné zařízení",
          "Nápo"
        ],
        "url": "https://www.facebook.com/permalink.php?story_fbid=" ...,
        "date": "2020-02-14T20:42:37.000Z",
        "canonical": "https://m.facebook.com/story.php?story_fbid=" ...
      }
    ],
    "average": 4.8,
    "count": 225
  },
  "services": [
    {
      "title": "The Refectory",
      "text": "In The Refectory "...
    }
  ],
  "title": "Hotel, Prague",
  "pageUrl": "https://www.facebook.com/...",
  "address": {
    "city": "Praha",
    "lat": 50.08905444,
    "lng": 14.40639193,
    "postalCode": "118 00",
    "region": "Prague",
    "street": "Letenská 12/33"
  },
  "awards": [],
  "email": "email@" ...,
  "impressum": [],
  "instagram": null,
  "phone": "+420 266 112 233",
  "products": [],
  "transit": null,
  "twitter": null,
  "website": "https://www.mar" ...,
  "youtube": null,
  "mission": [],
  "overview": [],
  "payment": null,
  "checkins": "11 504 lidí tu oznámilo svoji polohu",
  "#startedAt": "2020-03-31T17:26:01.919Z",
  "verified": true,
  "#url": "https://m.facebook.com/pg/...",
  "#ref": "https://www.facebook.com/.../",
  "#version": 1,
  "#finishedAt": "2020-03-31T17:34:22.979Z"
}
```

## Expected Consumption

One page and posts take around 1-2 minutes for the default amount of information (3 posts, 15 comments) to be generated, also depends on the proxy type used (`RESIDENTIAL` vs `DATACENTER`), block rate, retries, memory and CPU provided.

Usually, more concurrency is not better, while 5-10 concurrent tasks can finish each around 30-60s. A "20-concurrency" run can take up to 300s each. You can limit your concurrency by setting the `MAX_CONCURRENCY` environment variable on your actor.

A 2048MB actor takes an average `0.015` CU for each page on default settings. More "input page URLs" means more memory needed to scrape all pages.

**WARNING**: Don't use a limit too high for `maxPosts` as you can lose everything due to out of memory, or it may never finish. While scrolling the page, the partial content is kept in memory until the scrolling finishes.

Take into account the need for proxies that are included in the costs.

## Advanced Usage

You can use the `unwind` parameter to display only the posts from your dataset on the platform, as such:

```
https://api.apify.com/v2/datasets/zbg3vVF3NnXGZfdsX/items?format=json&clean=1&unwind=posts&fields=posts,title,pageUrl
```

`unwind` will turn the `posts` property on the dataset to become the dataset items themselves. the `fields` parameters makes sure to only include the fields that are important

## Limitations / Caveats

* Pages "Likes" count is a best-effort. The mobile page doesn't provide the count, and some languages don't provide any at all. So if a page has 1.9M, the number will most likely be 1900000 instead of the exact number.
* No content, stats or comments for live stream posts
* There's a known issue that some posts can make the crawler hang for a long time, using all the CPU. It's an edge case that involves a lot of variables to happen, but it's common to happen with a shared post from another live stream with links on both posts.
* New reviews don't contain a rating from 1 to 5, but rather is positive or negative
* Cut-off date for posts happen on the original posted date, not edited date, i.e: posts show as `February 20th 2:11AM`, but that's the edited date, the actual post date is `February 19th 11:31AM` provided on the DOM
* The order of items aren't necessarily the same as seen on the page, and not sorted by date
* Comments of comments (nested comments / conversations) aren't included in the output, only top-level comments on the posts.

## Versioning

This project adheres to semver.

* Major versions means a change in the output or input format, and change in behavior.
* Minor versions mean new features
* Patch versions mean bug fixes/optimizations (changes to `README.md` aren't tagged)

## Upcoming

* Separated dataset for posts, comments, and reviews

## License

Apache-2.0
