# Overview

This guide describes the basic architecture and fundamental abstractions of the
Carbyne Stack platform.

## Client/Server-style MPC

_Secure Multiparty Computation_ (MPC) is a cryptographic technique that
distributes a computation among multiple parties in such a way that no single
party can see the other parties' data.

Carbyne Stack implements SPDZ-like (see [1]) MPC in the client/server model,
first described by Damgård et al. in [2]. In this variant of MPC, computations
are offloaded by clients to a set of servers that act as the MPC parties.

Translated to the Carbyne Stack universe, this means, we can have a set of
Carbyne Stack deployments acting as the MPC parties[^1] and any number of
clients invoking the services provided collectively by the Carbyne Stack
deployments. In Carbyne Stack lingo, we refer to the MPC parties as _Virtual
Cloud Providers_ (VCPs) that jointly provide MPC services in a _Virtual Cloud_
(VC).

## Carbyne Stack Services

Each VCP hosts a set of elementary services called _Castor_, _Amphora_, and
_Ephemeral_ that together implement a fully functional cloud-native MPC party (
see Figure 1):

- **[Castor][castor]** stores data-independent tuples generated in the MPC
  offline phase[^2] and serves them on request to Amphora and Ephemeral.

- **[Amphora][amphora]** is the secret store that is used by clients to store
  and retrieve secrets. Secrets are stored as secret shares on the distributed
  Amphora instances in a VC. They can be used as inputs to an Ephemeral
  computation or are created as results of such a computation. Amphora uses
  data-independent tuples from Castor to facilitate secure up-/download in the
  client/server setting as described in [1].

- **[Ephemeral][ephemeral]** is used to execute programs using
  the [MP-SPDZ][mp-spdz] MPC framework. Any number of secrets can be fetched
  from Amphora at the beginning of an MPC program execution and any number of
  secrets can be written to Amphora at the end. Ephemeral fetches tuples from
  Castor consumed throughout the execution of the MPC program.

<figure>
  <img src="/components.png" width="600" />
  <figcaption>
    <b>Figure 1:</b> Carbyne Stack comprising the data store and
    compute services, clients, and a CLI
  </figcaption>
</figure>

## Communication Interfaces

The VCP services interact locally, with their counterparts in other VCPs, and
with clients (see Figure 2). There are three communication interfaces involved:

- The **Intra-VCP Interface** is used to communicate internally within a VCP.
  For example, Ephemeral talks to the co-located Castor service to fetch offline
  tuples using the intra-vcp interface[^3].

- The **Inter-VCP Interface** is used to coordinate operations among VCPs. An
  example for this is the interaction required between Amphora instances to
  perform the secure up-/download protocols mentioned above.

- The **Client Interface** is used by clients to invoke the Carbyne Stack
  services provided by a VC, e.g., uploading a secret to Amphora or triggering
  an MPC program execution via Ephemeral.

<figure>
  <img src="/interfaces.png" width="800" />
  <figcaption>
    <b>Figure 2:</b> The Carbyne stack components and the interfaces through
    which they communicate
  </figcaption>
</figure>

## Scalability

A central design goal of Carbyne Stack is that each of the VCP services can be
scaled independently and automatically. To achieve this, the Carbyne Stack
microservices are implemented as Kubernetes services (Castor and Amphora) and as
Knative applications (Ephemeral). This allows the scaling mechanisms of these
platforms to be used to react dynamically to fluctuating load patterns.

## Bibliography

[1] Ivan Damgård, Valerio Pastro, Nigel Smart, Sarah Zakarias: *Multiparty
computation from somewhat homomorphic encryption*. In: Safavi-Naini, R.,
Canetti, R. (eds.) Advances in Cryptology – CRYPTO 2012. Lecture Notes in
Computer Science, vol. 7417, pp. 643–662. Springer, Heidelberg, Germany, Santa
Barbara, CA, USA (Aug 19–23, 2012)

[2] Ivan Damgård, Kasper Damgård, Kurt Nielsen, Peter Sebastian Nordholt, Tomas
Toft: *Confidential Benchmarking based on Multiparty Computation*. In
Grossklags, J. and Preneel, B. (eds.) Financial Cryptography and Data Security,
pp.169–187. Springer.

[^1]: Only two-party settings are supported by Carbyne Stack as of now.
[^2]: Tuples must be generated externally and uploaded using the CLI as of
today. A future release of Carbyne Stack will include a service called *Klyshko*
to generate tuples in a scalable manner.
[^3]: Note that currently tuples baked into the container image are used by
Ephemeral. Fetching the tuples from Castor will be implemented in a future
release.

[castor]: https://github.com/carbynestack/castor

[amphora]: https://github.com/carbynestack/amphora

[ephemeral]: https://github.com/carbynestack/ephemeral

[mp-spdz]: https://github.com/data61/MP-SPDZ
