<div align="center">

![gogs-server-logo](https://user-images.githubusercontent.com/194400/164705537-8253ff09-581d-438d-8025-453eecb41b96.png "gogs server setup")

Deployment docs for our `Gogs` server on Fly.io: https://gogs-server.fly.dev

</div>

### Why `Gogs`?

We use GitHub as the
[single source of truth](https://en.wikipedia.org/wiki/Single_source_of_truth)
for our Product & Services. ⭐<br />
Occasionally GitHub has
["incidents"](https://www.githubstatus.com/history)
where it's offline for hours ... ⏳ <br />
Also, GitHub can
[**_lose_ data**](https://news.ycombinator.com/item?id=31033758)
if you're not careful;
i.e. **`delete`** is **_forever_**! 🤦‍♀‍<br />
So we needed an _easy_ way to **backup** our data. 💾

**`Gogs`** is a **_lightweight_ `Git` server**
with a familiar UI/UX
(think GitHub circa 2018 clone) <br />
that can be deployed in **5 minutes**
and has _most_ of the GitHub features we _use_. <br />
e.g:
Orgs, Repos, Markdown editor/viewer, Issues & Pull Requests.

### Why Fly.io?

Fly is a **_dramatically_ simplified** Platform as a Service
([PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service))
with an underlying easily accessible Infrastructure as a Service
([IaaS](https://en.wikipedia.org/wiki/Infrastructure_as_a_service)).
It combines the best elements of AWS and many from Heroku
but focusses on the essential and eliminates the bloat.
We love it and recommend it to anyone
tired of the _complexity_ of AWS or the _cost_ of Heroku.

### Why `Postgres`?

As the tagline says:
"_PostgreSQL: The World's Most Advanced Open Source Relational Database_."

We use and love it because it's fast, has excellent docs
and great tooling.

> **Note**: We've used MySQL or MariaDB in the past they are both good.
> We think Postgres is better.
> It's the default Database for
> [**`Phoenix`**](https://github.com/dwyl/learn-phoenix-framework#our-top-10-reasons-why-phoenix)
> our chosen Web Framework,
> so using it with our **`Gogs`** server makes sense
> to reduce cognitive load and cost.

<br />

## _What_?

This repository documents our deployment of our **`Gogs`** server.

See: https://gogs-server.fly.dev/

## How?

This is a step-by-step guide for recreating our server.
If you find it useful, please ⭐

### Create an App

Create a Fly.io App for the Gogs Server:

```sh
flyctl launch --name gogs-server --image gogs/gogs --org dwyl
```

> In our case we called our app `gogs-server`,
> pretty self-explanatory and not very creative.
> We like it when DevOps is
> [_immediately obvious_](https://en.wikipedia.org/wiki/KISS_principle).
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
<hr />

#### Quick Note on Fly.io Postgres Database Clusters

If you already have a Postgres database cluster on Fly,
you can host as many Postgres databases
on as you like, the resources are scaled up automatically.

> "_Users won't notice this!
> They’re directed to the nearest running instance automatically._"

Just remember to _enable_ autoscaling on your DB cluster (see below).

<hr />

### Create `PostgreSQL` DB & Attach to `Gogs`

Create a DB named `gogs-server-db` in the `lhr` (London) region:

```sh
fly pg create --name gogs-server-db --region lhr
```

> Use your preferred region.

### Enable Autoscaling

see: https://fly.io/docs/reference/scaling/

```sh
fly autoscale standard min=1
```

### Attach the DB to the `Gogs` App

Attach the DB to the `Gogs` server:

```sh
fly postgres attach --app gogs-server --postgres-app gogs-server-db
```

### Intialize `Gogs`!

When you first visit your `Gogs` instance,
you will be redirected to the `/install` page.
These were the settings we defined on ours:

![gogs-fly-config-1of-2](https://user-images.githubusercontent.com/194400/165000531-6e352107-860b-4bb2-ad29-4f172f6fc08d.png)

![gogs-fly-config-2of-2](https://user-images.githubusercontent.com/194400/165000528-79275762-f070-4bf0-b440-5a0fbf1ba9bc.png)

If you make a mistake with the setup of your Gogs server,
don't panic, you can _easily_ update
the
[`app.ini`](https://github.com/gogs/gogs/blob/main/conf/app.ini)
file to change any of the settings.

Login to the VM via ssh:

```sh
flyctl ssh console
```

Track down the `app.ini` file on the instance:

```sh
find / -name app.ini
```

In my case it was at:

```sh
/data/gogs/conf/app.ini
```

Before making any changes,
make a backup of the file
in case you need to revert.
e.g:
[`gogs-server/main/app.ini`](https://github.com/dwyl/gogs-server/blob/main/app.ini)

Edit/update it:

```sh
vi /data/gogs/conf/app.ini
```

I updated:

```sh
SSH_PORT = 22
```

To

```sh
SSH_PORT = 10022
```

This mirrors the port forwarding defined in the `fly.toml` file:
[fly.toml#L52](https://github.com/dwyl/gogs-server/blob/559d583070fe1db3d65189e662fefcb5932abc15/fly.toml#L52)

If you make any changes to the `app.ini` file,
you will need to restart the VM that is running your `gogs` instance.
e.g:

```sh
flyctl restart gogs-server
```

You will see output confirming the restart:

```sh
gogs-server is being restarted
```

### Test the `Gogs` Instance

https://gogs-server.fly.dev/nelsonic

![image](https://user-images.githubusercontent.com/194400/165000758-0ca9e54d-2c8a-429f-9c84-ff4bfd68bed2.png)

I created a couple of repos, one `public` the other `private` to test.

Next we want to _interact_ with a repo ...

<!--

### SSH Config

https://community.fly.io/t/ssh-connection-to-an-instance/834/2

```sh
2022-04-23T23:19:42Z   [info]2022/04/23 23:19:42 [FATAL] [gogs.io/gogs/internal/ssh/ssh.go:130 listen()] Failed to start SSH server: listen tcp 0.0.0.0:22: bind: permission denied
2022-04-23T23:19:43Z   [info]2022/04/23 23:19:43 [ INFO] Gogs 0.13.0+dev
```

-->

#### Add SSH Key

Add your `ssh` key to the `Gogs` instance
so that you can interact with the repo via `git`
in your terminal.

Copy the **_`public`_** `ssh` key on your main computer.
In my case the `id_rsa.pub` file is located at
`~/.ssh/id_rsa.pub`
on my Mac.
So to copy the contents of the file,
I run the following command:

```sh
pbcopy < ~/.ssh/id_rsa.pub
```

Next, connect to the `Gogs` Server
and visit the `/user/settings/ssh`
page, e.g:
https://gogs-server.fly.dev/user/settings/ssh

Once you've successfully added your **_`public`_** `ssh` key
to `Gogs` you should see a success message such as:

<img width="1217" alt="image" src="https://user-images.githubusercontent.com/194400/164786677-18901fe2-1c38-4419-b63f-395ba9ff6d9e.png">

#### Clone Repo

Create a repository if you don't already have one, e.g:
https://gogs-server.fly.dev/nelsonic/public-repo

![image](https://user-images.githubusercontent.com/194400/164581017-247d388b-0ed5-475a-960d-55140247e47c.png)

If you attempt to clone the repo
using a standard command, e.g:

```sh
git clone git@gogs-server.fly.dev:nelsonic/public-repo.git
```

You will see the following error:

```sh

ssh: connect to host gogs-server.fly.dev port 22: Connection refused
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
```

This is because the `TCP` port **`22`** is reserved
for actual `SSH` connections on Fly.io.
We could re-assign it for use with **`Gogs`**,
but then we would lose the ability to `ssh` into the instance ...
We don't want that,
because it's useful to **`fly ssh console`**
to debug & maintain the instance.

Attempt to specify the TCP port:

```sh
git clone -p 10022 git@gogs-server.fly.dev:nelsonic/public-repo.git
```

That doesn't work.
So reading:
https://stackoverflow.com/questions/5767850/git-on-custom-ssh-port

Trying the following format:
https://stackoverflow.com/a/5767880/1148249

```sh
git clone ssh://git@mydomain.com:[port]/org|usernam/repo-name.git
```

e.g:

```sh
git clone ssh://git@gogs-server.fly.dev:10022/nelsonic/public-repo.git
```

#### Make local changes

Update the `README.md` on my Mac:

<img width="1088" alt="image" src="https://user-images.githubusercontent.com/194400/165000134-7adfe672-01cf-4637-bb71-02c95e58709a.png">

#### Commit & Push Changes

`git commit` and `git push` the code:

```sh
git push
Enumerating objects: 5, done.
Counting objects: 100% (5/5), done.
Delta compression using up to 8 threads
Compressing objects: 100% (3/3), done.
Writing objects: 100% (3/3), 350 bytes | 350.00 KiB/s, done.
Total 3 (delta 1), reused 0 (delta 0), pack-reused 0
To ssh://gogs-server.fly.dev:10022/nelsonic/public-repo.git
   7f92c5d..f714a64  master -> master
```

#### Confirm they Worked!

https://gogs-server.fly.dev/nelsonic/public-repo
![image](https://user-images.githubusercontent.com/194400/165000085-de1166d1-6f89-4dc9-9c47-aae89fca9003.png)

Branches work:
![image](https://user-images.githubusercontent.com/194400/164999937-ac78290e-5d09-4016-8b26-252a1134ba98.png)

Here is the content on the `draft` branch:
![image](https://user-images.githubusercontent.com/194400/164999949-d0076a91-3cf5-417d-8944-82861e2e39d7.png)

## Recommended Reading

- Fly CLI: https://fly.io/docs/flyctl/
- Fly Launch: https://fly.io/docs/flyctl/launch/
- Fly Deploy: https://fly.io/docs/flyctl/deploy/
- App Configuration (fly.toml):
  https://fly.io/docs/reference/configuration/
- Appkata: Gogs - standalone Git Server (Fly.io)
  https://fly.io/docs/app-guides/git-gogs-server/
  **Note**: this is a bit old (2020)
  and there is no longer a `fly init` command
  and the `fly.toml` file is incomplete.
  Hence us needing to write these docs.
  Thanks given to @codepope in:
  https://community.fly.io/t/gogs-standalone-git-service-as-a-fly-example/358/2
- Multi-region PostgreSQL:
  https://fly.io/docs/getting-started/multi-region-databases/
- Scaling and Autoscaling
  https://fly.io/docs/reference/scaling/
- Autoscale:
  https://fly.io/docs/flyctl/autoscale/
- Auto-scaling tutorial:
  https://hosting.analythium.io/auto-scaling-shiny-apps-in-multiple-regions-with-fly-io/
- SSH troubleshooting:
  https://docs.github.com/en/authentication/troubleshooting-ssh/using-ssh-over-the-https-port

<hr />

[![HitCount](http://hits.dwyl.com/dwyl/gogs-server.svg)](http://hits.dwyl.com/dwyl/gogs-server)
