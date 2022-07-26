[![LinkedIn][linkedin-shield]][linkedin-url]

[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/teague-stockwell/

<br />
<p align="center">
  <a href="https://hello-next-auth.vercel.app">
    <img src="./public/logo-120-120.png" alt="Logo" width="120" height="120">
  </a>

  <h3 align="center">Buildable</h3>

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
    <li><a href="#database-migrations">Database Migrations</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
</details>

## About The Project

A post is a way to share the build process of something that contains modular parts. For example a post about a bike would contain wheels, tires, a sprocket, a handlebar, pedals etc...

These parts should be listed on the post so users can explore other posts by filtering based on a specific part or parts

A user that's want to build a similar bike uses these posts to find part combinations that make their ideal bike

Buildable is a collection of multiple sub reddits that contain posts and their parts. One could be about bikes, and another could be Baking

### API

The api is deployed as serverless functions to vercel - I believe this uses AWS lambda under the hood. These functions use an ORM called Prisma to query a managed MySql cluster hosted on planetscale. The api service is partially RESTfull. Later I wrote a [middleware for serverless functions](https://github.com/teaguestockwell/next-api-mw) that made the apis functionality composable in a very similar way to react hooks. After this, some of the endpoints started drifting from RESTfull to more of a RPC pattern. Eventually, I would like to use the paid versions on vercel and planetscale to rollout a multiAZ deployment, but for now the free tier of each has been working great.

### UI

Using nextjs and react most of the UI is served from a CDN as static html that gets updated with the latest data every so often. When the browser loads the app, the client shows that stale html while it fetches the latest from the server in tha background.
Parts of the UI that are not public read are rendered on the server - like the page that edits a post. In either case the client is hard at work combining the server's content with rich client side functionality.
Some notable parts of the UI are:

- A component library called Mantine
- React query for caching and fetching server state
- Zustand for client state
- Emotion for css in js

### Scraping service

The scraping service will allow users to add urls to parts they used in their post. The bulk of the scraping service is currently being built as an open source library [here](https://github.com/teaguestockwell/price-scraper). It uses chromium to load a page then parses structured data embedded in the html similar to how [google indexes products for rich results](https://developers.google.com/search/docs/advanced/structured-data/intro-structured-data). This seems to work on on retailers that support a format of structured data, but in the future there may need to be something less fragile and generic like a natural language processing pipeline to extract product information. Whether or not this could be cost prohibitive because of compute remains tbd.

### Authentication

The serverless functions use a package called Next Auth to authenticate users with googles oauth service. From their each function validate that a request has been authenticated, and then proceed to authorizing the request.

### Serverless Connection Pooling

Serverless functions quickly exhaust db connections. A few of the solutions I have tried include:

- AWS RDS and RDS Proxy
- A PgBouncer on an EC2 instance
- Digital Ocean with PostgreSQL and PgBouncer.

After working with these, I switched to MySQL and use planetscale to host a managed vitese cluster. planetscale did a few things very well that set it a step ahead ov the above solutions:

- A free tier with up to 1000 connections
- Pull request based schema changes
- Excellent built in logs
- Multi AZ deployments
- Auto scaling to the moon because of vitese

### Image Uploads

1. The client authenticates with Googles Oath service
1. The client sends a requests a presigned upload url
1. A serverless function rate limits the request using a sliding log algorithm
1. The serverless function creates a record of the pending image that has not been assigned a resource
1. The serverless function responds to the client with a presigned upload url
1. The client attaches a picture to a post or community
1. The client compresses the picture using a service worker until it meets the size limit on the pre signature
1. The client uploads the picture using th pre signature
1. The client submits the form containing the picture to the server
1. A serverless function verifies the picture was uploaded correctly by sending a head request to s3
1. The serverless function deletes the pending upload job
1. The serverless function runs a chron job to cleanup stale pictures

### Incremental Static Regeneration & React Query

Next.js can generate static HTML incrementally into it's edge network cache. When that page is served to the client, the next data is passed to react-query as as initial data. From there react-query can take over and do some powerful things like refetch on window focus and optimistic updates for mutations.

### Threaded comments

[full writeup](https://teaguestockwell.com/blog/threaded-comments)

- statically cached html
- client side stale while revalidate
- optimistic updates

### Search

A user may search for new communities to follow, and other users. This is currently implemented using a serverless functions, a db query, and a client cache. In the future I would like to learn more about elastic search and scale this out into a separate search service.

## Built With

- [TypeScript](https://www.typescriptlang.org)
- [React](https://reactjs.org/)
- [Nextjs](https://nextjs.org/docs/api-reference/create-next-app)
- [Prisma](https://www.prisma.io/)
- [Next Auth](https://next-auth.js.org/)
- [React Query](https://react-query.tanstack.com/)
- [Zustand](https://github.com/pmndrs/zustand)
- [Mantine](https://mantine.dev/)

## System Architecture

- Planet Scale's Vitess cluster service
- AWS S3 Bucket behind the AWS Cloudfront CDN for user images
- Vercel's serverless functions and edge network
- Google Oauth

<p align="center">
  <img src="https://user-images.githubusercontent.com/71202372/180944745-01102da8-404f-4de1-8538-6b8f94cec2ec.png" alt="system architecture" width="1000vh" >
</p>

## Database Migrations

Prisma.schema is the source of truth for data models. To deploy changes to production:

- update prisma schema
- run `npx prisma generate` to generate the typescript api for this schema locally
- run `npm run docker:db:migrate` to generate the sql migration file
- [install the planetscale cli](https://github.com/planetscale/cli) `brew install pscale`
- authenticate pscale `pscale auth login`
- push schema changes to a new db branch `pscale branch create buildable branch-name-here`
- connect to the branch `pscale connect buildable branch-name-here --port 3309`
- push sql migration to the branch `DATABASE_URL="mysql://root@127.0.0.1:3309/buildable?connect_timeout=30&pool_timeout=30&socket_timeout=30" npx prisma db push`
- create a pr `pscale deploy-request create buildable branch-name-here`
- merge pr with cli or web ui `pscale deploy-request deploy buildable 1` 1 is the pr number reported by the cli when the pr was created

### Troubleshooting migration failures

1. Error: P1000: Authentication failed against database server at `xxx`, the provided database credentials for `xxx` are not valid.
   - go to the web ui for that branch and manually create a new connection string instead of using the local proxy.

## Roadmap

See the [open issues](https://github.com/teaguestockwell/buildable-readme/issues) for a list of proposed features (and known issues).

## Contact

Teague Stockwell - [LinkedIn](https://www.linkedin.com/in/teague-stockwell)

## Acknowledgements

- [RotorBuilds](https://rotorbuilds.com/)
- [Lee Robinson's AWS upload template](https://github.com/leerob/nextjs-aws-s3)
