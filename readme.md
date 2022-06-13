[![LinkedIn][linkedin-shield]][linkedin-url]

[linkedin-shield]: https://img.shields.io/badge/-LinkedIn-black.svg?logo=linkedin&colorB=555
[linkedin-url]: https://www.linkedin.com/in/teague-stockwell/

<!-- PROJECT LOGO -->
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

<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary><h2 style="display: inline-block">Table of Contents</h2></summary>
    <li><a href="#about-the-project">About The Project</a></li>
    <li><a href="#system-architecture">System Architecture</a></li>
    <li><a href="#database-migrations">Database Migrations</a></li>
    <li><a href="#roadmap">Roadmap</a></li>
    <li><a href="#contact">Contact</a></li>
    <li><a href="#acknowledgements">Acknowledgements</a></li>
</details>

<!-- ABOUT THE PROJECT -->

## About The Project

A post is a way to share the build process of something that contains modular parts. Some examples of use cases would be software, food, bikes, and [fpv quadcopters](https://rotorbuilds.com/)

For example a post about a bike would contain wheels, tires, a sprocket, a handlebar, pedals etc...

These parts should be listed on the post so users can explore posts by filtering based on a specific part or parts

A user that's want to build a similar bike uses these posts to find part combinations that make their ideal bike

Parts should be categorized. In the example of the bike, handlebars would be a one category, frames would be another

Some interesting features of buildable that have been implemented are:

- A threaded commenting system
- Client side image compression with [browser-image-compression](https://github.com/Donaldcwl/browser-image-compression)
- Rich text editing with [quill](https://github.com/quilljs/quill)
- Client side image uploads with presigned urls and rate limiting
- Like / save posts and like comments
- Optimistic updates for likes, saves, and comments with React Query and rollbacks on server error
- Pages are initially loaded from cached incremental statically generated HTML with Next.js and Vercel, then the data is refetch client side with React Query
- Search topics (sub reddits), and users

### Serverless Connection Pooling

Initially I used AWS RDS, but the serverless functions quickly exhausted all the connections to my small RDS instance. One possible solution is using AWS RDS Proxy to pool connections on postgres. Unfortunately, because of the way RDS proxy pins connections it cant be used with [prisma](https://www.prisma.io/docs/guides/deployment/deployment-guides/caveats-when-deploying-to-aws-platforms). Another way may be to roll a custom PgBouncer on an EC2 instance. I ended up switching to Digital Ocean because they offered a managed PostgreSQL with PgBouncer. Finally I switched to Planet Scales hosted Vitess cluster of MySQL DBs.

### Presigned Image Uploads

When a client want to upload an image, a request is sent to the API to return a presigned upload URL. Before the API returns the URL, it can make sure the user is signed in and that they have not exceeded their rate limit. In the future, rate limiting would be handled by Redis, but for now it lives in a DB table. Once the client receives the upload URL any they input an image, it gets compressed in the browser using [browser-image-compression](https://www.npmjs.com/package/browser-image-compression) until it is within the upload limit. When the image is submitted to the API, the API sends a HEAD req to verify the upload was successful. It can then save the image and remove the Pic job it created when issuing the presigned URL to track stale objects that may be in S3.

### Incremental Static Generation & React Query

Next.js can generate static HTML incrementally into it's edge network cache. When that page is served to the client, the next data is passed to react-query as as initial data. From there react-query can take over and do some powerful things like refetch on window focus and optimistic updates for mutations.

### Authentication

Next Auth and Google OAuth are used to authenticate users. From there the API can determine what roles a user has.les a user has.

## Built With

- [Create Next App](https://nextjs.org/docs/api-reference/create-next-app)
- [TypeScript](https://www.typescriptlang.org)
- [Next Auth](https://next-auth.js.org/)
- [Prisma](https://www.prisma.io/)
- [Zustand](https://github.com/pmndrs/zustand)
- [Ant Design](https://ant.design/)
- [React](https://reactjs.org/)
- [React Query](https://react-query.tanstack.com/)

## System Architecture

- Planet Scale's Vitess cluster service
- AWS S3 Bucket behind the AWS Cloudfront CDN for user images
- Vercel's serverless functions and edge network
- Google Oauth

<p align="center">
  <img src="https://user-images.githubusercontent.com/71202372/138579764-eca2f1fb-8f8e-43df-9d6a-f56d5dc5f788.png" alt="system architecture" width="1000vh" >
</p>

## Database Migrations

Prisma.schema is the source of truth for data models. To deploy changes to production:

- update prisma schema
- run `npx prisma generate` to generate the typescript api for this schema locally
- run `npm docker:db:migrate` to generate the sql migration file
- [install the planetscale cli](https://github.com/planetscale/cli) `brew install pscale`
- authenticate pscale `pscale auth login`
- push schema changes to a new db branch `pscale branch create buildable branch-name-here`
- connect to the branch `pscale connect buildable branch-name-here --port 3309`
- push sql migration to the branch `DATABASE_URL="mysql://root@127.0.0.1:3309/buildable?connect_timeout=30&pool_timeout=30&socket_timeout=30" npx prisma db push`
- create a pr `pscale deploy-request create buildable branch-name-here`
- merge pr with cli or web ui `pscale deploy-request deploy buildable 1` 1 is the pr number reported by the cli when the pr was created

## Roadmap

See the [open issues](https://github.com/teaguestockwell/buildable-readme/issues) for a list of proposed features (and known issues).

## Contact

Teague Stockwell - [LinkedIn](https://www.linkedin.com/in/teague-stockwell)

## Acknowledgements

- [RotorBuilds](https://rotorbuilds.com/)
- [Lee Robinson's AWS upload template](https://github.com/leerob/nextjs-aws-s3)
