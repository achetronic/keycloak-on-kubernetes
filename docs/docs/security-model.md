# Security Model

## Motivation

> One of the major security risk for all systems is not located on the source code itself but most times on the people 
> who can access them.

## Diagram

![sso model diagram](./img/sso-model.png)

## 🔒 People with access

Everyone must assume the amount of people who have access to this system should keep as minimal as possible to
cover several areas:

- Maintenance and security: As the past demonstrated in the company, places where everyone is touching at the same time, 
  are systems that rapidly gets dirty and unmaintainable. Dirty systems tends to reduce the security over the time and 
  this is something not ignorable in this system.
- Legal: Some laws (GDPR, LOPD) ask the companies to store the user data into a hardened place. Moreover, these data
  and the ways they can be reached must be trackable. Tracking these things is not easy when everyone can touch the system.

Because of that, we expose the decisions we have made about the people with privileges, and the reasons in the following
lines:

- **CTO:** The maximum position about engineering stuff is the legal responsible for the data leaks, so the role have to
  enforce hardening of this system over the time.
- **COE:** This role is directly linked to the same responsibilities mentioned for CTO. So this role must enforce this kind
  of policies about hardening the security over the time.
- **SRE Chapter lead:** SRE members are the people who directly manage and maintain this system. The mission for this role 
  is to build and enforce directly the security policies.
- **1-2 SRE members:** SREs should not stay alone. We are all humans and can make mistakes. SREs here has the mission to
  discuss and participate in the construction of the security policies. This position should always be filled with someone with
  seniority as SRE

This documentation is getting hard to read, uh! so let's put some emojis here 🎉🎉🎉🤓🤓

## What is a realm

If you are a developer, you should know the concept of _tenancy_. In the basis, a realm is like a tenancy for Keycloak.
Each realm support its own users, its own clients and its own everything. There is only one limitation: as described in
the official documentation, even when the `master` realm can do everything, it should only be used to create and manage
secondary ones, where the actual life is happening.

For this reason, as you can see in the diagram on top of the page, we have a `master` realm where only administrators
can enter, and other secondary realms to solve the homogenization and security problems of the company:

- **Internal:** This realm is intended only for internal tools usage. Vault, Gitlab, etc, should be protected using clients
  of this realm. All employees are registered on this one.
- **Customers:** This realm has a clear purpose to store all the customers of the company. The clients of this realm must
  be the applications that access the customers' data

## What is a client

A client represents an application which needs to access user data, for example, getting a JWT. In the case of this
company, an example could be a dashboard. The dashboard gets JWTs for the signed-in users and use them to call the APIs.
Those JWTs have some personal user data about profile or permissions, so the dashboard should be considered as an application
which needs a Keycloak client.

If you still don't understand the concept, the key idea is that a client is like a challenge the application have to
perform to get some user data in the form of a JWT.

## How to get a client

The problem about everyone managing the realms' clients is that, most times, they include credentials like the 
client `id` or `secret`, and this could represent a security risk for the company.

One of the related tasks for infrastructure people is to reduce the exposed surface for key services like this one,
so we as SRE, thought about a flow to manage the clients in a safe way. which is represented in the following diagram:

```text
    ┌────────────────────┐     ┌─────────────────────────────┐      ┌───────────────────────────┐
    │  Developers craft  ├─────►  Developers ask SRE member  ├──────►  SRE members evaluate     │
    │  an application    │     │  for a realm client         │      │  possible security risks  │
    └────────────────────┘     └─────────────────────────────┘      └────────────┬──────────────┘
                                                                                 │
                                                                                 │
    ┌────────────────────┐     ┌─────────────────────────────┐      ┌────────────▼──────────────┐
    │ Application uses   │     │ SRE store the credentials   │      │ SRE create a suitable     │
    │ the credentials    ◄─────┤ inside right path on Vault  ◄──────┤ client for the use case   │
    └────────────────────┘     └─────────────────────────────┘      └───────────────────────────┘
```

As you can see, SRE and developers working together, as DevOps philosophy dictates. Because remember, DevOps is NOT
a position but a mentality.
