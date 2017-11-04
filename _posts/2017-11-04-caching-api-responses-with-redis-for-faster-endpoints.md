---
layout: post
title: "Caching API responses with Redis for faster endpoints"
excerpt: "Redis is an in-memory database project implementing a key-value store with optional durability"
tags: [nodejs, javascript, redis, cache, api, typescript]
date: 2017-11-04T10:14:08+02:00
comments: true
image:
  feature: posts/cover-redis.png
---

Have you ever implemented an API endpoint that takes a while to respond? Maybe some seconds?

Imagine when you have thousands of clients accessing your API at the same time and this response time increases because of that.

An I/O call such as reading content from a file or a very complicated query in the database may take a while to complete. If this content doesn't change at all, it's much better then to leave it in the RAM, as accessing something there will be much faster than accessing something that is on disk. If it changes from time to time, a script to update the cache can be implemented to be called via a cron job when needed and the endpoint code can be refactored to fetch data only from the cache.

In this post I expect you to have Node.js and Redis already configured.

## Redis

Taken from their website: Redis is an open source (BSD licensed), in-memory data structure store, used as a database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries, bitmaps, hyperloglogs and geospatial indexes with radius queries.

As Redis is a database that stores everything in memory, the access to it is lightning fast! Much faster than accessing something from disk. This is perfect for what we need.

## Use case

This code was developed for Node.js in TypeScript and uses Redis, Bluebird, async/await and a database file from [Maxmind](https://www.maxmind.com/en/geoip2-databases) for the lookup of IP related data (not included in the example).
This is an example of blocking the content or part of it according to the IP address that was detected, comparing it with the regions that are allowed to receive that response (geo-targeting).

If the IP address is not from one of the configured regions, the code would filter the response and respond with only the parts of it that were configured for its region. In order to do that, we need to know all the regions in the country, so they can be compared with the region that was detected for the IP address of the request.

Inside the config directory I have the configuration for Redis inside the cache.ts file:

{% highlight typescript linenos %}
const log = require("pino")();
import * as bluebird from "bluebird";
const redis = bluebird.Promise.promisifyAll(require("redis"));

const cacheAddress = process.env.CACHE_HOST || "127.0.0.1";
const cachePort = process.env.CACHE_PORT || 6379;

let cache = redis.createClient(cachePort, cacheAddress);

if (process.env.CACHE_AUTH === "true") {
    cache.auth(process.env.CACHE_PASS);
}

cache.on("error", (err) => {
    if (err.message.indexOf("ECONNREFUSED") !== -1) {
        log.error("Error: The server was not able to reach Redis. Maybe it's not running?");
        process.exit(1);
    } else {
        throw err;
    }
});

export default cache;
{% endhighlight %}

Below is an example of a class with a method to load the regions for a country.

The regions need to be loaded in order to be compared with the region of the IP address. If it is not in the cache, the method loads the file from disk into cache, so for the next request it will be serving from cache already (The return statement makes sure the rest of the code is not executed if the content is cached).

{% highlight typescript linenos %}
import * as fs from "fs";
import * as util from "util";
import cache from "../config/cache";
const readFile = util.promisify(fs.readFile);

class GeoIP {

    public filterAllowedInstances = async (instances: Array<IConfigResponse>, ipAddress: string): Promise<any> => {
        try {
            if (instances.length === 0) return [];

            let allowedInstances = [];

            let lookup = this.ipLookup(ipAddress);
            let ipGeonameId = lookup.subdivisions[0].geoname_id;

            for (let instance of instances) {
                let response = await this.isIPAllowed(ipGeonameId, instance.regions);
                if (response) allowedInstances.push(instance);
            }

            return allowedInstances;
        } catch (err) {
            if (err.message === "Local ip" || err.message === "Wrong database") {
                return instances;
            }
            Validation.logErrorInternal("GeoIP", err);
            return [];
        }
    }

    private isIPAllowed = async (ipGeonameId: number, instanceRegions: Array<string>): Promise<Boolean> => {
        if (!instanceRegions || instanceRegions.length === 0) return true; // No region is configured, so allow all

        if (this.countryRegions.length === 0) { // Load all the regions for the country
            this.countryRegions = await this.loadRegionsForCountry();
        }

        for (let region of instanceRegions) {
            let filteredRegion = this.countryRegions.filter(regionAll => regionAll.name === region);
            if (filteredRegion.length === 0) continue;
            if (filteredRegion[0].geoname_id === ipGeonameId) return true;
        }

        return false;
    }

    private loadRegionsForCountry = async (): Promise<any> => {
        let cachedRegions = await cache.getAsync("regions");
        if (cachedRegions) return JSON.parse(cachedRegions);

        let countries = await readFile(`${__dirname}/../config/country_region_data.json`, "utf8");
        const parsedCountries = JSON.parse(countries);
        const countryCode = process.env.COUNTRY_CODE;
        const regions = parsedCountries.filter(code => code.countryShortCode === countryCode.toUpperCase())[0].regions;

        await cache.setAsync("regions", JSON.stringify(regions));
        return regions;
    }

}
{% endhighlight %}

If thousands of requests would need to have their IP addresses filtered against regions that were in a file on disk or in the database, then it would take a lot of file reading or calls to the database for that, so when the information comes from RAM it's much faster.

Redis methods usually don't have the "Async" suffix and use callbacks, but because of bluebird's promisifyAll method, these methods with this suffix were created with a Promise implementation.

## Conclusion

This was an example of loading a file into Redis, so it can be served from cache instead of from the disk. You can use this idea for your project with some other cases, for example when viewing a statistics page, where the backend code needs to calculate many things before responding so the frontend can show it. In this case, the results could be cached on Redis and updated from time to time via a cron job. This way when the user opens the statistics page, the page would load instantly instead of making him wait every time for the calculation.

Do you have another interesting example, any doubt or suggestion? Leave a comment :)
