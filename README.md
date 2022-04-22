# `gogs-server`

CI/CD Pipeline + Docs for our Gogs Server on Fly.io

## Setup App

```
flyctl launch --name gogs-server --image gogs/gogs --org dwyl
```

```sh
? Select configuration:  [Use arrows to move, type to filter]
> Development - Single node, 1x shared CPU, 256MB RAM, 1GB disk
  Development - Single node, 1x shared CPU, 512MB RAM, 10GB disk
  Production - Highly available, 1x shared CPU, 256MB RAM, 10GB disk
  Production - Highly available, 1x Dedicated CPU, 2GB RAM, 50GB disk
  Production - Highly available, 2x Dedicated CPU's, 4GB RAM, 100GB disk
```

I went with `Development` for now.

<!-- "init" is not a valid command! and Error unknown flag: --port
```sh
flyctl init gogs-server --image gogs/gogs --org dwyl --port 3000
```
-->

Create a Volume (Network Attached Storage):
https://fly.io/docs/reference/volumes/

```sh
fly volumes create data --region lhr
```

```sh
fly postgres attach --app gogs-server --postgres-app gogs-server-db
```
