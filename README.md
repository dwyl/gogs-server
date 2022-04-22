<div align="center">

![gogs-server-logo](https://user-images.githubusercontent.com/194400/164705537-8253ff09-581d-438d-8025-453eecb41b96.png "gogs server setup")

Deployment docs for our `Gogs` server on Fly.io.

</div>

### Why `Gogs`?

We use GitHub as the
[single source of truth](https://en.wikipedia.org/wiki/Single_source_of_truth)
for our Product & Services.
Occasionally GitHub has
"incidents" https://www.githubstatus.com/history
where it's offline for hours ... ⏳
Also, GitHub can
[**_lose_ data**](https://news.ycombinator.com/item?id=31033758)
if you're not careful; i.e. **`delete`** is **_forever_**!
So we needed an _easy_ way to **backup** our data.

Gogs is a lightweight Git server
that can be deployed in **5 minutes**
and has _most_ of the GitHub features we use.

### Why Fly.io?

Fly is a **_dramatically_ simplified** Platform as a Service
([PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service))
with an underlying easily accessible Infrastructure as a Service
([IaaS](https://en.wikipedia.org/wiki/Infrastructure_as_a_service)).
It combines the best elements of AWS and many from Heroku
but focusses on the essential and eliminates the bloat.
We love it and recommend it to anyone tired

### Why `Postgres`?

## How?

### Create an App

Create a Fly.io App for the Gogs Server:

```sh
flyctl launch --name gogs-server --image gogs/gogs --org dwyl
```

> In our case we called our app `gogs-server`,
> pretty self-explanatory and not very creative.
> We like it when DevOps is _obvious_.
> It dramatically reduces cognitive overload
> and context switching costs!

Select the type of instance you want:

```sh
? Select configuration:  [Use arrows to move, type to filter]
> Development - Single node, 1x shared CPU, 256MB RAM, 1GB disk
  Development - Single node, 1x shared CPU, 512MB RAM, 10GB disk
  Production - Highly available, 1x shared CPU, 256MB RAM, 10GB disk
  Production - Highly available, 1x Dedicated CPU, 2GB RAM, 50GB disk
  Production - Highly available, 2x Dedicated CPU's, 4GB RAM, 100GB disk
```

We went with `Development` for now,
but once we have everything setup
we will return and create a `Production` instance.

### Create a Volume

Create a Volume (Network Attached Storage):
https://fly.io/docs/reference/volumes/

```sh
fly volumes create data --region lhr
```

The volume is called `data`.
But under-the-hood the fly system
gives it a unique name.

You can easily check this if needed by running:

The default size is 10Gb.
We definitely won't need that much.

<br />

## Quick Note on Fly.io Postgres Database Clusters

If you already have a Postgres database cluster on Fly,
you can host as many Postgres databases
on as you like, the resources are scaled up automatically.

> "_Users won't notice this!
> They’re directed to the nearest running instance automatically._"

Just remember to _enable_ autoscaling on your DB cluster (see below).

### Create `PostgreSQL` DB & Attach to `Gogs`

Create a DB named `gogs-server-db` in the `lhr` (London) region:

```sh
fly pg create --name gogs-server-db --region lhr
```

> Use your preferred region.

### Enable Autoscaling

### Attach the DB to the `Gogs` App

Attach the DB to the `Gogs` server:

```sh
fly postgres attach --app gogs-server --postgres-app gogs-server-db
```

### Intialize `Gogs`!

> Insert screenshots here!

### Test the `Gogs` Instance

#### Add SSH Key

>

#### Clone Repo

>

#### Make local changes

>

#### Commit & Push Changes

>

#### Confirm they Worked!

>

## Recommended Reading

- Fly CLI: https://fly.io/docs/flyctl/
- Fly Launch: https://fly.io/docs/flyctl/launch/
- Fly Deploy: https://fly.io/docs/flyctl/deploy/
- Appkata: Gogs - standalone Git Server (Fly.io)
  https://fly.io/docs/app-guides/git-gogs-server/
  **Note**: this is a bit old (2020)
  and there is no longer a `fly init` command
  and the `fly.toml` file is incomplete.
  Hence us needing to write these docs.
- Multi-region PostgreSQL:
  https://fly.io/docs/getting-started/multi-region-databases/
- Scaling and Autoscaling
  https://fly.io/docs/reference/scaling/
