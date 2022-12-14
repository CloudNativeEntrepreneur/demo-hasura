# demo-hasura

Auth Example's Hasura instance, configs, migrations, and job to run migrations.

This particular example relies on SchemaHero to manage the "readmodel" database. However, if you'd like to use Hasura to manage this DB's migrations, simply uncomment the following lines:

https://github.com/CloudNativeEntrepreneur/demo-hasura/blob/master/Dockerfile#L7
https://github.com/CloudNativeEntrepreneur/demo-hasura/blob/master/apply-migrations.sh#L6

## Environment Variabls

### `HASURA_GRAPHQL_ENDPOINT`

Using this env when running Hasura console will override the value in `config.yaml`.

```
HASURA_GRAPHQL_ENDPOINT=http://some-endpoint hasura console
```

To get the Hasura url, run `kubectl get ksvc` in the namespace.

```
NAME                           URL                                                                LATESTCREATED                        LATESTREADY                          READY   REASON
demo-hasura                https://demo-hasura.jx-staging.cloudnativeentrepreneur.com                 demo-hasura-00008                demo-hasura-00008                True   
```

### Model services

To add a new model service as an available env variable for Hasura actions, edit `charts/demo-hasura/templates/ksvc.yaml`

## What the Chart creates

This chart provisions a PSQL cluster with two databases, `hasura` owned by the user `hasura`, and `sourced` owned by the user `sourced`, as well as a knative serving service that runs the hasura-graphql-engine.

# Local Development with local-dev-cluster

For local development, the easiest way to use Hasura is to rely on the database in the local development cluster, while running Hasura through docker-compose - this was Hasura can send Actions to your local microservices, but still have the database created and maintained via Schemahero.

To do so:

1. Make sure `local-dev-cluster` is running: https://github.com/CloudNativeEntrepreneur/local-dev-cluster

This will set up Postgres Operator and Schemahero.

2. Deploy a Database and Schema to your local dev cluster: https://github.com/CloudNativeEntrepreneur/example-readmodel

In that repo, the command `make onboard` is configured to deploy the db to your local cluster.

3. "Port Forward" the database so it's accessible locally

In a new terminal window, as you need to leave the process running, run:

```
kubectl port-forward example-readmodel-postgresql-0 5433:5432
```

4. Start Hasura in this repo against port-forwarded db

Run `make up` to start Hasura via docker-compose configured to connect to the port-forwarded database. This command runs a script which looks up the required password for the db and provides it to the `docker-compose.yaml` file.

5. When adding actions in HASURA, first add an ENV VAR to the docker-compose file with the URL, using `host.docker.internal` instead of `localhost` as the host. Using ENV Vars is not required, but makes switching to production versions simple later on.

It's worth noting that you could port-forward any psql db running in any kubernetes cluster. It does not have to be the local-dev-cluster, or the example-readmodel db, however, development being simple is important, and hence I've preconfigured this solution for development purposes.

# FAQ

## YOUR SECRET IS EXPOSED

Yes, I know.

It'll only work in the local development cluster - this is part of an example that contains several moving parts, so I just generated some random secrets where they were needed and preconfigured things accordingly so you can just run it locally and everything will work. Couldn't go without secrets as part of that example is an authentication server and it's JWT integration with Hasura.

Don't use these proconfigured values in production. I typically use ExternalSecrets in prod.
