# Keycloak

## Description

Deployment manifests for Keycloak inside Kubernetes.

> Keycloak is a provider for identities (IdP) and the authorization flows needed to identify them (OIDC)

## Diagram

This system is in charge of the security of other systems, so it lives alone, with a lot of care, but alone. The following
diagram only shows how it interacts with other kind of services.

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
                          │  APIs  │               │   GUI   │
                          └────────┘               └─────────┘
```

You may be interested in a detailed insight about the system and how interacts with other system. Of course, if you
are that crazy, you can launch the full documentation (which is exposed in the meta-documentation) following
[these steps](./README.md#full-documentation) and see the full diagrams inside the section `Security Model`

## Motivation

There are two scenarios where this has been needed:

1. The companies commonly handle several internal tools which need to be authenticated or authorized to allow entrance.
   such as a secrets vault or a pipelines executor. Even when most them expose ways to access, these ways are heterogeneous
   and uncomfortable. For convenience, a way to reduce overhead for employees is enabling SSO (Single Sign On) on every
   tool that is capable to use it.

2. Companies products need different ways to authorize its usage: SDKs, APIs, etc; and usually craft
   a different authorization flow for each product due to some of them are bought, others are homemade, and developers
   need to code fast and reduce team interdependency. In some point everything usually become a mesh for everyone and it is
   needed to unify these flows for all the company, allowing each tool to be authorized in a different way, but
   using the same user pool (cough, cough!! do you remember Oauth?)

## Requirements

> **ATTENTION**
>
> All the requirements are already covered by other projects crafted by us:
>
> - [Tooling Stack](https://github.com/prosimcorp/tooling-stack)
> - [Monitoring Stack](https://github.com/prosimcorp/monitoring-stack)

To deploy this system inside Kubernetes, some requirements must be satisfied.

Requirements depends on the environment you need to deploy. For example, to deploy using `production` manifests,
there are some extra dependencies that are not needed when deploying using `develop` ones.

To make this section easy to understand, choose an environment and read the following list:

### Commonly used

To expose the system to external traffic, the following ones are required:

- [Ingress NGINX controller](https://kubernetes.github.io/ingress-nginx/)
- [Cert Manager](https://cert-manager.io/)
- [External DNS](https://github.com/kubernetes-sigs/external-dns)

### Required only on `production`

- [External Secrets](https://external-secrets.io/)

## How to deploy

> **The deployment will be done assuming `develop` environment**. If you are trying to deploy this in production,
> some extra requirements must be satisfied, better explained on the [requirements section](./README.md#requirements)

Deploying this project is easy, just follow the following commands from the root path of the repository:

### Deploy the namespaces where everything will live

> Namespaces are deployed apart due to some automation tools like FluxCD, delete the resources deleting directly the
> namespaces. So we would like to have the namespaces safe if some day the infrastructure cluster is automated again.

```console
kubectl apply -k ./deploy/dependencies/namespaces
```

### Deploy the database

First you have to deploy the operator we use to deploy the database:

```console
kubectl apply -k ./deploy/dependencies/postgresql/postgres-operator
```

Then you can deploy the database

> Credentials for this step are get from Vault, so remember than Vault must be working first

```console
kubectl apply -k ./deploy/dependencies/postgresql/crs/prepare/develop
kubectl apply -k ./deploy/dependencies/postgresql/crs/cluster/develop
```

### Deploy Keycloak

This is the easy part, there we go:

```console
kubectl apply -k ./deploy/resources/develop
```

> When deploying Keycloak, an admin user can be created. This is suitable for first starts and for undesirable situations
> where some administrator simply meshed up everything and it is needed to restore the order and peace into Keycloak.
> For doing this, just uncomment desired environment variables inside the `ConfigMap` done for that purpose on deployment
> manifests. Don't use this in daily scenarios.

## Full documentation

Full documentation is available as a complement for this one. To know more, and how to, there are a
[specific guide](./docs/README.md) you can follow with simple steps for that.

## How to contribute

You can do the magic in just a few steps:

1. Fork the repository
2. Create a branch and do the changes you need
3. Open a pull request, filling all the fields correctly, you know, a good description, squash your commits, etc.
4. Wait until the PR is approved by the team
5. Ready to merge the code, so drink beers, cheers and enjoy 🍻 🎉 🎉
