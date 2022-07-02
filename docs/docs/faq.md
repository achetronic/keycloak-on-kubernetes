# FAQ

## 🪲 How to debug the problems with the system?

Due to all the systems are deployed inside Kubernetes, what about watching logs before coming to this documentation?

Inside the Infrastructure cluster:

```console
aws eks --region eu-west-1 --profile production update-kubeconfig --name infrastructure
```

See the logs?

```console
kubectl logs -f -n keycloak statefulset.apps/keycloak
```

---

## Why is deployed as StatefulSet?

Keycloak pods are more or less stateless. They are clustered using Infinispan under the hoods, to be as much scalable
as desired. The key here is that not all the information is stored distributed, but some data, are stored locally in the pods,
for example, session data, which means if you start the auth flow with a node, only that node stores the data.

This mechanism is strong, because if some node does not know some data, simply asks other nodes for it, reducing the 
performance. Do you imagine a lot of nodes spamming others for the session data?

Keycloak includes a mechanism to distribute some data that are defaulted to be local, so we chose to distribute the sessions
too. And the number of nodes which store those data has to be set manually inside a `ConfigMap`, so we decided to launch
the system as a `StatefulSet` to make others understand that there is a cluster under the hoods, with minimal manual 
intervention, but not auto-managed.

Variables involved are `CACHE_OWNERS_COUNT: "3"` and `CACHE_OWNERS_AUTH_SESSIONS_COUNT: "3"`

---

## Why the master realm is empty?

It is used for management purposes of other realms. This is a good practise established by RedHat in the Keycloak
official documentation. We just accommodate to it.

---

## 😨 I destroyed the system. What do I do?

This is one of the first systems you have to launch, doe to several ones can be accessed through this. By the moment,
take a breath and think slower.

This system is easy to manage, because everything depends only on a few points. The most important one is to distinguish
if you have a fully functional EKS cluster, with the [Tooling Stack]() and [Monitoring Stack]() installed inside.

This is important because, by the moment, Infrastructure cluster is the only one in the company managed completely manually, 
due to an inception problem when automating the deployments using FluxCD in the same destination cluster that is the source 
of truth too (you can imagine the problem)

Once you have confirmed it, the next step is to launch the Postgres. You have the deployment manifests in the repository
and the instructions in the README. Basically, launch the operator we use for PostgreSQL, and then the database.

I can imagine what you are thinking right now: _"Hey, I need to restore the data, bro"_

We have covered you on this exact situation, so kiss us later, keep reading...

The clusters with the Tooling Stack deployed inside, cover the `VolumeSnapshot` Kubernetes API,
and have an operator to fully automate these snapshots. The snapshots are stored with the `Retain` policy.

The idea is that you have snapshots on AWS for the volumes provisioned for this PVC (in fact, for almost of infra related ones)

Now you are more relaxed, the instructions:

1. **Find the snapshots** related to this PVC in AWS. You can allocate them using the tags. 
   The tags always should have enough information for these moments.
2. From snapshots, **you are interested only in the `id`** parameter
3. Go to the directory with the manifests for disaster recovery situations for PostgreSQL.
   **Modify the manifests inside** `crs/cluster/base/disaster-recovery/VolumeSnapshotContents` to include the ID of the 
   snapshots. As you can see, the manifests are auto-documented to help you today.
4. Uncomment the `disaster-recovery` line in the Kustomization to launch the cluster:
   `crs/cluster/base/kustomization.yaml`
5. Modify the PVC manifests to start from the snapshot and not to be provisioned again:
   `crs/cluster/base/persistentVolumeClaims`, again, they are auto-documented
6. Deploy the cluster again, as the **README** says. Probably you want to skip the `prepare` stage for Postgres. It is only 
   there to get the credentials from Vault. Don't worry if you see the operator detects that the Postgres cluster is not ready, 
   or whatever, it is because of the credentials not matching. 

> Focus, first finish this disaster recovery, and you will have Keycloak running again. 
> The next systems to recover and check should be Gitlab and Vault, because these three affects to the rest of the 
> products. 

> **A hint: Gitlab disaster recovery is done in the same way**

---

Cheers! 🍻🍻🍻
