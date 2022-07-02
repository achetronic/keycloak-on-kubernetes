# Overview

## Motivation

There are two scenarios where this has been needed:

1. The company handle several internal tools which need to be authenticated or authorized to allow entrance. such as
   secrets vault or pipelines executor. Even when most them expose ways to access, these ways are heterogeneous and
   uncomfortable. For convenience, SRE team is trying to reduce this overhead for our customers, the developers, enabling
   SSO (Single Sign On) on every tool that is capable to use it.

2. Several company products need different ways to authorize its usage: SDKs, APIs, etc. In the past, the company crafted
   a different authorization flow for each product, due to some of them were bought, others were homemade, and developers needed to
   code fast and reduce team interdependency. This kind of practises are producing security risks and technical debt for 
   the future ([better described in this section](./security-model.md)). In some point everything became a mesh for everyone and SRE 
   team decided to help there, unifying these flows for all the company, allowing each tool to be authorized in a different 
   way, but using the same user pool (cough, cough!! do you remember Oauth?)

For these reasons, SRE team decided to test a lot of tools for the same purpose (Supertokens, Ory Kratos + Hydra, etc)
and in the end, Keycloak won the battle due to the low maintenance footprint, existence of a well-documented API and
the maturity of the project on every aspect, for instance, security matter.

## Current stack

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
are that crazy and want to know this stuff better, you can go to the [apropiate section](./security-model.md#diagram)

## Where is deployed

This system is deployed inside the **infrastructure** Kubernetes cluster (on AWS, production account). So if you have
some kind of problem, now you know where to look into.

You can read the deployment documentation in the **README** of the repository:

- [Keycloak](https://gitlab.infrastructure.s73cloud.com/Infrastructure/keycloak)
