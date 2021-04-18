Status: In Development

The extension registry needs to implement the [server API](https://github.com/adobe/brackets/wiki/Extension-Repository-Server-API). We want the registry to be:

* fast
* simple
* reliable

*Fast* because speed is a feature that users appreciate, *simple* because we really want to spend more of our time building Brackets itself rather than extension infrastructure and *reliable* because we don't want our users to be thwarted in their attempts to install extensions. We also want reliability in terms of not losing our extension data.

The complete registry will be built in two parts:

* The *repository*, which is responsible for providing Brackets users with the extension listing information as well as the extension packages themselves
* The *registry*, which is responsible for managing the repository

The initial version of the registry will provide an web-based interface for uploading packages. Extension authors will authenticate using OAuth with GitHub.

## Simplicity: Node-based Registry Server on EC2 ##

Node offers a number of features that make it well-suited to serving as the platform for the API:

* [Passport](http://passportjs.org/) is a straightforward authentication/authorization library with easy OAuth support and plugins for many services
* Support for [zip files](https://github.com/springmeyer/node-zipfile), [gzip](http://nodejs.org/api/zlib.html) and [tar](https://github.com/isaacs/node-tar)
* Implementing HTTP+JSON APIs is the bread and butter for node
* Ability to share code for package validation and possibly other uses between Brackets and the server
* Everyone on the team knows JavaScript already :)
* Node's single-threaded nature makes it straightforward to synchronize writes to the registry

The registry server code will be in the [brackets-registry repository](https://github.com/adobe/brackets-registry). We have chosen [Express](http://expressjs.com/) as the web framework and will use [Handlebars templates](https://github.com/donpark/hbs).

## Speed and Reliability: S3-based Repository ##

S3 is reliable, durable, inexpensive and scalable in both speed and size. By serving our extension metadata and extension packages directly from S3, installing extensions and keeping them up to date will be reliable and immediate.

S3 supports HTTPS, but not with custom domain names. Brackets will retrieve information directly from our S3 bucket over HTTPS to ensure that no one uses a man-in-the-middle attack on extension installers.

Note: AWS does not support gzipping of files directly. We'll want to be sure to gzip the extension metadata. Upload a gzipped file and set the header appropriately.

## Database? ##

Given node's single-threaded nature, we can just keep the extension listing as an in-memory data structure. We write it out to disk (likely saving old versions just-in-case) and to S3 periodically.

User registrations *might* warrant a database, but it may be just as easy to work with the user data in the same manner as the extension listing. When the user database consists just of Brackets extension authors, that will clearly be a small amount of data. Even when we add ratings and reviews, only a fraction of Brackets users will use these features.

For reviews and ratings, we can pull the detailed info from S3 (probably keeping a cache on disk to avoid eventual consistency issues) and just store them directly in that JSON data. No need to keep it in memory.

If the authoritative data for everything is stored in S3, then we will not need to bother with EBS or using one of Amazon's database services. If the API server goes down, it needs only to reload from S3 to get started.

## Log Processing ##

The system described above would rely on S3 logging to keep track of download counts for each extension. Periodically, those logs would need to be processed and download counts updated. Depending on how much traffic there is to wade through, we may not want the API server node process to be responsible for this, because it would not be able to respond to requests while the logs are being processed.

A separate log processor process (which would likely be written in Node, but could be written in anything) would rip through the log files and update download counts through APIs in the node server added specifically for this purpose.