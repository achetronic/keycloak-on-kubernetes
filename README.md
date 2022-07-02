# Keycloak

## Description

Deployment manifests for Keycloak inside Kubernetes.

> Keycloak is a provider for identities (IdP) and the authorization flows needed to identify them (OIDC)

## Diagram

This system is in charge of the security of other systems, so it lives alone, with a lot of care, but alone. The following
diagram only shows how it interacts with other projects.

```text
                       ┌───────────────────────────────────────────────────┐
                       │                                                   │
                       │                                                   │
                       │     ┌┬────────────┬┐          ┌┬──────────┬┐      │
                       │     ││ PostgreSQL │◄──────────►│ Keycloak ││      │
                       │     └┴────────────┴┘          └┴──────────┴┘      │
                       │                                                   │
                       │                                                   │
                       └───────▲────────────▲───────────▲─────────▲─Keycloak
                               │            │           │         │
                               │            │           │         │
                               │            │           │         │
                               │            │           │         │
                               │        ┌───▼───┐       │       ┌─▼─────┐
                               │        │ Vault │       │       │ . . . │
                               │        └───────┘       │       └───────┘
                          ┌────▼───┐               ┌────▼────┐
                          │ Dremel │               │ Polynet │
                          └────────┘               └─────────┘
```

You may be interested in a detailed insight about the system and how interacts with other system. Of course, if you
are that crazy, you can launch the full documentation (which is exposed in the meta-documentation) following 
[these steps](./README.md#full-documentation) and see the full diagrams inside the section `Security Model`

## Motivation

There are two scenarios where this has been needed:

1. The company handle several internal tools which need to be authenticated or authorized to allow entrance. such as 
   secrets vault or pipelines executor. Even when most them expose ways to access, these ways are heterogeneous and 
   uncomfortable. For convenience, SRE team is trying to reduce this overhead for our customers, the developers, enabling 
   SSO (Single Sign On) on every tool that is capable to use it.

2. Several company products need different ways to authorize its usage: SDKs, APIs, etc. In the past, the company crafted
   a different authorization flow for each product due to some of them were bought, others were homemade, and developers needed to
   code fast and reduce team interdependency. In some point everything became a mesh for everyone and SRE team decided 
   to help there, unifying these flows for all the company, allowing each tool to be authorized in a different way, but
   using the same user pool (cough, cough!! do you remember Oauth?)

For these reasons, SRE team decided to test a lot of tools for the same purpose (Supertokens, Ory Kratos + Hydra, etc)
and in the end, Keycloak won the battle due to the low maintenance footprint, existence of a well-documented API and
the maturity of the project.

## Requirements

To deploy this system inside Kubernetes, several tools are required. They are already covered by the
[Tooling Stack](https://gitlab.infrastructure.s73cloud.com/Infrastructure/tooling-stack) and the 
[Monitoring Stack](https://gitlab.infrastructure.s73cloud.com/Infrastructure/monitoring-stack)

## How to deploy

Deploying this project is easy, just follow the following commands from the root path of the repository:

### Deploy the dependencies

1. Deploy the namespaces where everything will live:

> Namespaces are deployed apart due to some automation tools like FluxCD, delete the resources deleting directly the 
> namespaces. So we would like to have the namespaces safe if some day the infrastructure cluster is automated again.

```console
kubectl apply -k ./deploy/dependencies/namespaces
```

2. Deploy the database:

First you have to deploy the operator we use to deploy the database:

```console
kubectl apply -k ./deploy/dependencies/postgresql/postgres-operator
```

Then you can deploy the database

> Credentials for this step are get from Vault, so remember than Vault must be working first

```console
kubectl apply -k ./deploy/dependencies/postgresql/crs/prepare/production
kubectl apply -k ./deploy/dependencies/postgresql/crs/cluster/production
```

### Deploy Keycloak

This is the easy part, there we go:

```console
kubectl apply -k ./deploy/resources/production
```

> When deploying Keycloak, an admin user can be created. This is suitable for first starts and for undesirable situations 
> where some administrator simply fuc*@$? up everything and you need to restore the order and peace into Keycloak.
> For doing this, just uncomment desired environment variables inside the `ConfigMap` done for that purpose on deployment
> manifests. Don't use this in daily scenarios.

## Full documentation

Full documentation is available as a complement for this one. To know more, and how to, there are a 
[specific guide](./docs/README.md) you can follow with simple steps for that.

## How to contribute

You can do the magic in just a few steps:

1. Create a branch and do the changes you need 
2. Open a pull request, filling all the fields correctly, you know, a good description, squash your commits, etc.
   > Due to the risk of changes on this repository, the reviewer for PRs must be an experienced 
   > [SRE member](https://gitlab.infrastructure.s73cloud.com/groups/SRE/-/group_members)
3. Wait until the PR is approved by the SRE team
4. Ready to merge the code, so drink beers, cheers and enjoy 🍻 🎉 🎉
