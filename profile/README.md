# Steem

### Team members:

- Gabriela Limberea
- Robert Bălan

---

## General description

Steem is an online game shopping platform meant to provide publishers/developers a
space where they can showcase and sell their games. At the same time, it serves as a
game library, allowing users to purchase, play, and even return games. Anyone can
browse through the catalogue of games available on the platform and check out what
piques their interest.

### Types of users and what they can do

**Non-authenticated user:**

- can list all games available
- can inspect a game in detail

**Authenticated user:**

- same as non-authenticated user
- can add money to their account
- can check their account balance
- can see their own personal library
- can buy games with money
- can return games within a 48 hours time frame from the time of purchase,
given that the user has played the game at most for 2 hours
- can play a game

**Publisher/developer user:**

- same as non-authenticated user
- can see all their published games (approved, pending or rejected)
- can publish/update/delete a game
- can check their profits

**Admin user:**

- can list all games on the platform (approved, pending or rejected)
- can inspect all games on the platform
- can approve/reject a publishing request
- can list all users

---

## Application architecture

![Arhitecture Diagram](https://github.com/steem-idp-project/.github/blob/main/profile/img/architecture-diagram.png)

### [Auth API](https://github.com/steem-idp-project/auth-api)

This microservice is responsible for handling user registration and access. When a user
registers or logs in, the API provides them with a session cookie under the form of a
JWT token. Also, it exposes an endpoint for token validation, which is used
mainly by the Steem API. All interactions with the database are done through the IO
API.

*Tech stack*: python (flask)

## [Steem API](https://github.com/steem-idp-project/steem-api)

The core component of the architecture, it handles all previously mentioned types of
user actions. It interacts with both the Auth API (for authorization checks) and the IO
API (for honoring user requests) through HTTP requests. Here is where request content
validation is done.

*Tech stack*: python (flask)

### [IO API](https://github.com/steem-idp-project/io-api)

The API exposes HTTP endpoints as wrappers over the standard CRUD operations
with the PostgreSQL database. At this level, no data validation will be done, any errors
thrown by the database are forwarded back to the business logic API or the Auth API.
The schema for the database looks like this:

![Database schema](https://github.com/steem-idp-project/.github/blob/main/profile/img/db-schema.png)

*Tech stack*: python (flask, psycopg2)

### [Platform infrastructure](https://github.com/steem-idp-project/platform-infra)

This is the repository with all the deployment scripts and manifest files for
the platform. The deployment is thought to be done on a 3-node Kubernetes
cluster, with the following components:

- Postgresql database, using the **Cloud Native Postgresql Operator**
- Pgadmin4 for direct database management
- Portainer for Kubernetes cluster management
- Prometheus + Grafana for cluster monitoring and alerting
- Steem API, Auth API and IO API deployments
- Nginx gateway, using the **Nginx Ingress Controller**, for access to Grafana,
  Pgadmin4, Portainer and the APIs, using the following subdomains:
  - *idp.admin-steem.com* for Grafana, Pgadmin4 and Portainer
  - *api.steem.com* for the APIs
- An additional **Github Actions Runner** deployment, to allow us to run CI/CD
  pipelines that can interact with the cluster. All pipelines are thought to be
  triggered by the creation of a new release tag on the repositories, and are
  responsible for building, pushing and deploying the new images to the cluster.

Local development was done using **Kind**.
