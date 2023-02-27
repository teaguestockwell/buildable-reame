[![LinkedIn][linkedin-shield]][linkedin-url]

[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/teague-stockwell/

<p align="center">
  <a href="https://hello-next-auth.vercel.app">
    <img src="./public/logo-120-120.png" alt="Logo" width="120" height="120">
  </a>
  <h3 align="center">More Builds</h3>
  <p align="center">
   A social media platform for exploring and sharing buidable items
    <br />
    <a href="https://hello-next-auth.vercel.app">View Site</a>
    ·
    <a href="https://github.com/teaguestockwell/buildable-readme/issues">Report Bug</a>
  </p>
</p>

<details open="open">
  <summary><h2 style="display: inline-block">Table of Contents</h2></summary>
    <li><a href="#about-the-project">About The Project</a></li>
    <li><a href="#system-architecture">System Architecture</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
</details>

## About The Project
- share the build process of something that contains modular parts. for example a bike contains wheels, tires, a sprocket etc...
- users can explore other buids by filtering based on a specific parts
- more builds is a collection of topics that contain builds parts and their retialers.

## System Architecture
<p align="center">
  <img src="https://user-images.githubusercontent.com/71202372/221341449-aecfc595-09d4-4871-a713-cb2a3df0e8dd.jpg" width="1000vh" >
</p>

- gcp oauth
    - the browser perform an authentication flow, then it uses the api to [exchange an acsess code for an jwt cookie](https://sourcegraph.com/notebooks/Tm90ZWJvb2s6MTc2MQ==)
- vercel cdn
    - serve static acsess like html css and js to the browser
- vercel sererless functions
    - stateless deployment of a backend service that exposes crud, rate limiting, authorization, rbac and presigned upload urls (sas urls)
- aws cdn 1 + bucket 1
    - store and serve original imgs uploads
- aws cdn 2 + bucket 2
    - store and serve size variants of the origianl blobs
- aws functions
    - optimaize orignial imgs into many sizes with [sharp](https://aws.amazon.com/blogs/networking-and-content-delivery/image-optimization-using-amazon-cloudfront-and-aws-lambda/)
- planetscale mysql
    - a managed rdbs for horizontally scaling a cluster of mysql databases
- github next-api-mw
    - repo / feed for api middleware packakge [next-api-mw](https://www.npmjs.com/package/next-api-mw)
- github price-scraper
    - repo / feed for [getting product data such as price, name, images](https://www.npmjs.com/package/price-scraper)
- github more-builds
    - a repo for a monolithic app service with nextjs
- github scraper
   - a repo for a docker img built with express / nodejs / chromium for web scraping
- azure container registry
   - a artifact feed for docker images waiting to be applied to k8s
- azure k8s cert manager
   - rotates tls certs from lets encrpt
- azure k8s control plane
   - scraping service orchestration
- azure k8s ingress
   - https ingress and load balancer
- azure k8s nodes
   - nodejs scraper instances

### UI
- using nextjs and react most of the UI is served from a CDN as static html that gets updated with the latest data every so often
- when the browser loads the app, the client shows that stale html while the js fetches the latest data from the server in tha background.
- the ui used a mix of client and server side rendering to show contentfull content quickly
- in either case the client is hard at work combining the server's content with rich client side functionality.


### API
- deployed as serverless functions to vercel
- these functions use an orm called prisma to query a managed mysql cluster hosted on planetscale
- the api service is stateless and consumed using remote procedure call (RPC) and a [middleware i wrote for serverless functions](https://github.com/teaguestockwell/next-api-mw) that makes api functionality composable in a similar way to react hooks

### Scraping service
- allow users to add urls to parts they used in their post
- the bulk of the scraping service is built as an open source library [here](https://github.com/teaguestockwell/price-scraper). 
- it uses chromium to load a page then parses structured data embedded in the html similar to how [google indexes products for rich results](https://developers.google.com/search/docs/advanced/structured-data/intro-structured-data).
- [full writeup](https://teaguestockwell.com/blog/price-scraper)


### Serverless Connection Pooling
- serverless functions quickly exhaust db connections: a few of the solutions I have tried include:
  - aws rds and rds proxy
  - pgbouncer on an ec2 instance
  - digital ocean with postgres and pgbouncer.
- after working with these, I switched to planetscale to host a managed vitese cluster because it provides:
  - a free tier with up to 1000 connections
  - pull request based schema changes
  - excellent built in logs
  - multi AZ deployments
  - auto scaling to the moon because of vitess
  
### DB
- notably, when deploying to vitess, fk containtraints are not enforced
  - [this mean that fk indices need to be manually added](https://www.briananglin.me/posts/spending-5k-to-learn-how-database-indexes-work/)
```mermaid
erDiagram

  User {
    String email PK 
    String userId  
    String name  
    DateTime createdAt  
    DateTime updatedAt  
    String profileSrc  "nullable"
    String profilePicS3Key  "nullable"
    String about  "nullable"
    Int profileViews  
    Int postViews  
    Int numFollowers  
    }
  

  Post {
    String postId PK 
    String email  
    String topicId  
    String topicName  
    String userId  
    DateTime createdAt  
    DateTime updatedAt  
    Int numLikes  
    String html  
    Int views  
    Json storePartsQty  "nullable"
    }
  

  Comment {
    String commentId PK 
    String email  
    String postId  
    String parentCommentId  
    DateTime createdAt  
    DateTime updatedAt  
    Int numLikes  
    String text  
    }
  

  Pic {
    String picId PK 
    String email  
    String postId  "nullable"
    String partId  "nullable"
    String topicId  "nullable"
    String s3Key  
    DateTime createdAt  
    }
  

  PicJob {
    String s3Key PK 
    DateTime createdAt  
    }
  

  RateLimit {
    String rateLimitId PK 
    String email  
    DateTime createdAt  
    Int resource  
    }
  

  Topic {
    String topicId PK 
    String name  
    String about  
    DateTime createdAt  
    DateTime updatedAt  
    Int followerCount  
    Int views  
    }
  

  PartCategory {
    String partCategoryId PK 
    String topicId  
    String topicName  
    String name  
    String about  
    }
  

  Part {
    String partId PK 
    String topicId  
    String partCategoryId  
    Int numPosts  
    String name  
    String description  "nullable"
    String src  "nullable"
    }
  

  Store {
    String storeId PK 
    String name  
    }
  

  StorePart {
    String storePartId PK 
    String partId  
    String storeId  
    String price  
    String url  
    DateTime updatedAt  
    }
  
    User o{--}o Post : ""
    User o{--}o Post : ""
    User o{--}o Topic : ""
    User o{--}o User : ""
    User o{--}o User : ""
    User o{--}o Topic : ""
    Post o{--|| User : "user"
    Post o{--|| Topic : "topic"
    Post o{--}o User : ""
    Post o{--}o User : ""
    Post o{--}o StorePart : ""
    Comment o{--|| User : "user"
    Comment o{--|| Post : "post"
    Comment o{--}o User : "liked"
    Comment o{--}o User : "saved"
    Pic o{--|o Post : "post"
    Pic o{--|| User : "user"
    Pic o|--|o Topic : "topic"
    RateLimit o{--|| User : "user"
    Topic o{--}o User : ""
    Topic o{--}o User : ""
    PartCategory o{--|| Topic : "topic"
    Part o{--|| Topic : "topic"
    Part o{--|| PartCategory : "partCategory"
    StorePart o{--|| Store : "store"
    StorePart o{--|| Part : "part"
    StorePart o{--}o Post : ""
```

### Image Uploads
```mermaid
sequenceDiagram
participant bucket
participant api2
participant cdn
participant service worker
participant browser
participant api1
participant db

note over bucket, db: cleanup imgs never commited
opt chron job
api1 ->> db: get jobs where expired
db -->> api1: jobs
api1 ->> bucket: delete imgs
bucket -->> api1: ok
api1 ->> db: delete jobs
db -->> api1: ok
end

note over bucket, db: resized img cache miss
opt cache miss
cdn->>api2: resize img {key} {size}
api2->> bucket: get {key}
bucket -->> api2: img {key}
api2 ->> api2: resize
api2 ->> bucket: upload resized
bucket -->> api2: ok
api2 -->> cdn: cache hit
end

note over bucket, db: service worker cache miss
opt cache miss
service worker -->> cdn: get img {key} {size}
cdn --> service worker: img {key} {size}
end

note over bucket, db: img query
browser ->> service worker: get img {key} {size}
service worker -->> browser: img {key} {size}

note over bucket, db: img upload
browser ->> api1: get presigned url
api1 ->> db: rate limits where user, resourse, time
db -->> api1: rows
api1 ->> api1: rate limit using a sliding log algorithm
api1 ->> api1: sign an upload for size and url with private key
api1 ->> db: create job
db -->> api1: ok
api1 -->> browser: presigned url
browser ->> browser: select and crop an img
browser ->> service worker: resize img
service worker ->> service worker: compress
service worker -->> browser: compressed img
browser ->> bucket: upload img with progress
bucket -->> browser: ok
browser ->> api1: commit upload {key}
api1 ->> bucket: verify {key}
bucket -->> api1: ok
api1 -->> db: delete job
db -->> api1: ok
api1 -->> browser: commited
```

### Incremental Static Regeneration & React Query
```mermaid
sequenceDiagram
participant client
participant cdn
participant api

note over client, db: incremental static generation
client ->> cdn: get html
opt cache miss
cdn ->> api: get html
api ->> db: get data
db -->> api: data
api ->> api: render html
api -->> cdn: html
end
cdn -->> client: html with stale data
client ->> client: first contentful paint
client -->> client: react hydrates
client -->> client: query cache populated with stale data

note over client, db: stale while revalidate
client -->> api: get fresh data
api -->> client: fresh data
client -->> client: diff data
opt is diffrent
client ->> client: render  data
end

note over client, db: client side rendering
client ->> api: get data
api -->> client: data
client ->> client: render data

note over client, db: server side rendering
client ->> api: get page
api ->> db: get data
db -->> api: data
api ->> api: render html - server side rendering
api -->> client: html with fresh data
client ->> client: first contentful paint
client --> client: react hydrates
```

### Threaded comments
- [full writeup](https://teaguestockwell.com/blog/threaded-comments)
- statically cached html
- client side stale while revalidate
- optimistic updates

### Search
a simple database query with a client cache

## Roadmap
See the [open issues](https://github.com/teaguestockwell/buildable-readme/issues) for a list of proposed features (and known issues).

## Contact
Teague Stockwell - [LinkedIn](https://www.linkedin.com/in/teague-stockwell)

## Acknowledgements
- [TypeScript](https://www.typescriptlang.org)
- [React](https://reactjs.org/)
- [Nextjs](https://nextjs.org/docs/api-reference/create-next-app)
- [Prisma](https://www.prisma.io/)
- [Next Auth](https://next-auth.js.org/)
- [React Query](https://react-query.tanstack.com/)
- [Zustand](https://github.com/pmndrs/zustand)
- [Mantine](https://mantine.dev/)
