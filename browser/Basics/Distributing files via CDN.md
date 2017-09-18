Distributing files via CDN

CDN (Content Delivery Network) is a special technology that allows a visitor to receive content from different geographical locations.

Why do I need a CDN?

Imagine that your site is used by people from the US and Russia. If your servers are in Russia, then for your visitor from Russia your site will work faster. For a person from the US - slower. This is due to the fact that in the first case both the visitor and the server are not far from each other. And in the second case, they are separated by the ocean. And for such large distances, there is a delay in data transmission. This leads to the fact that the speed of the site will be different for different locations.

To solve this problem, there is a CDN. The decision itself is rather trivial. In order for a person from the United States to get the content of the site faster, you need to transfer this content to the US. Those. We just add the server to the right places and copy the content of our site. Structure of CDN

Who needs it?

It makes sense to connect CDN only if your site is designed for an audience that can be located at a significant distance from the server (thousands of kilometers).

What exactly should I give via CDN?

It makes sense to use only those resources that do not change often, but are often requested:

Pictures
Javascript
CSS
How does this work in practice?

There are a large number of services that provide CDN services. The developer only needs to determine the set of files that will be available from different places and transfer this set of files to the system.

Most often, CDNs work in passive mode. Those. You do not need to transmit anything. You just specify the address of the original server in the delivery system settings. Example CDN Javascript

In HTML you specify a path not to your server, but to the CDN server:

<script type="text/javascript" src="/javascript.js">
<script type="text/javascript" src="http://cdn.somecdn.com/javascript.js">
# The exact path can be found from the CDN provider

Service Providers

The most popular providers of content delivery networks are:

CloudFlare . A large network, there are free services.
MaxCDN . Flexible prices for large and small sites.
The most important

CDN can significantly increase the speed of your site for visitors who are far from the hosting space. But apply it only if there are a significant number of such people.