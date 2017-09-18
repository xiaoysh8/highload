Optimizing CDN Costs with Amazon Route 53

Usually, CDNs are used when a site or application has an audience spread over a large territory. However, at some point in time CDN will start to cost a lot (because you always pay for traffic). In this case, it is not necessary to build your CDN or to sacrifice user experience for the sake of economy.

Recall that CDN - it's just a network of computers that give content to visitors to the site: Visitors send a request to CDN

How much is that

Now the server of medium configuration, for example in the Selector, costs about $ 250 per month. Typically, these servers have a bandwidth of 100MB and a fairly large limit on monthly traffic (sometimes even without it).

This means that for $ 250 you can service the return of approximately 5 ... 10 million photos per day (about 1 TB, with an average photo size of 250 KB).

At the same time, if you use Cloudfront for the same task, maintenance of 30 TB per month will cost10 times more expensive.

Using a partial distribution

CDN will be very disadvantageous if your audience is distributed, but has concentration points. For example:

Germany: 50%
New York: 40%
Other locations: 10%
In this case it will be reasonable:

Deliver servers that deliver content in Germany and New York.
For queries from other geographies, use the CDN.
Those. have a structure in which some of the requests are served by the CDN, and some are our servers: Distribution of requests between CDNs with their own servers

It is necessary to take into account that CDN providers have increased availability indicators. Sometimes, it may be more profitable to overpay and stay with a customized quality system. In other cases, you can provide a simple and reliable solution yourself using, for example, several data centers in each location.

Example implementation based on Amazon Route 53

To implement the scheme, we will need a Geo DNS provider, for example Amazon Route 53 . The solution scheme is as follows:

You must create a new subdomain on the primary domain. At us it will be testi.onthe.io.
For it, you need to create a CNAME record of Geolocation type for all default locations. It should point to the subdomain of your CDN provider:
Configuring GeoDNS in amazon route 53
Now for the locations in which our servers are located, you need to create separate CNAME records. In the example, an entry for North America:
Configuring GeoDNS in amazon route 53
# Instead of "CDN Provider" should be (sub) domain CDN provider, increase

Now the domain testi.onthe.io will return the content as follows:

If a visitor requests content from North America, it will be distributed immediately from the domain i.onthe.io (our servers).
If the content is requested from another country, the request will be served by the CDN provider.
And this means that we will not pay CDN provider for traffic, which we can serve and do it more efficiently.

TL; DR

The cost of CDN can be significantly reduced if your audience has concentration points. Then for a part of the audience it will be possible to give content from their servers, and for another part - with the help of CDN. To do this, you will need to rent several servers and use Geo DNS .