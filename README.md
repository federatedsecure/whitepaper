Federated Secure Computing: Technical Whitepaper
================================================

Authors:
Ballhausen, H (LMU Munich)

# Abstract

## Background and Motivation

Privacy-preserving computation (PPC) enables computation on several parties’ proprietary data that cannot be shared openly between participants. The family of technologies includes secure multiparty computation, differential privacy, homomorphic encryption, and others.

In practice, there are often steep barriers to entry. Small and medium enterprises, startups and individuals, research and teaching institutions lack the specific knowledge and resources to implement solutions and deploy and maintain the demanding and complex tech stacks.

Federated Secure Computing is an open-source initiative championed by Ludwig-Maximilians-Universität München (LMU Munich) and funded by Stifterverband. The project aims to provide access to PPC to a larger developer audience without specialist knowledge. 

## Methods

Federated Secure Computing features a client/server architecture. Its main design goal is to render client-side development of business logic as simple as possible.  Server-side cryptography is hidden behind a lean API. Data flow and control flow are both streamlined:

Each data owner operates their own server. This enables clients to send data to their proprietary server in clear without the need to generate cryptographic shares client-side. Alternatively, proprietary data bases may be directly accessed server-side with control flow from the client.

Control flow happens through microservices provided by an Open API 3.0 interface. A lean middleware represents server-side objects to the client and forwards requests to the cryptographic backend. Microservices wrap fine-grained cryptographic instructions into higher level macros to provide simpler and tested routines to non-expert client-side developers.

## Results

A free and open-source implementation is provided at github.com/federatedsecure

The server-side implementation is 100% Python with no tech stack needed beyond a standard webserver such as Connexion or Django (both are provided as stubs). The client-side implementation is just a thin API wrapper, available in several programming languages. A simple multiparty computation (“Simon”) protocol is provided as propaedeutic example of a cryptographic backend. 

## Conclusion

Federated Secure Computing is a free and open-source software (FOSS) initiative funded by Stifterverband and maintained by LMU Munich. The architecture aims to render PPC more accessible and inclusive, as a first steppingstone to build privacy-friendly applications. We hope this whitepaper enables anyone to join the ecosystem, and we are welcoming contributions.

# Background

# Methods (Architecture)

## Pain Points and Design Goals

Presently, PPC still has several barriers to entry. Some are one the business side (lacking financial incentives, unproven business models) ore purely historical (lack of visible role models and successful show cases). These are out of the scope. However, many barriers are related to rather technical pain points that render development, deployment, and operation of PPC solutions cumbersome and difficult. These can and should be addressed by the architecture.

The following pain points are recurring topics in the literature, have been encountered in interviews with prospective early adopters, or have been experienced firsthand by the authors. Each has led to the formulation of a specific design goal addressing those concerns:

|     pain points                                                                                           |     design goals                                                                                      |
|-----------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
|     ** related to capabilities (C) **                                                                     |                                                                                                       |
|     front-end and business logic developers   rarely have any expert knowledge in PPC                     |     provide high-level computing routines that hide low-level cryptography   protocols (C1)           |
|     PPC is   inaccessible to marginal groups lacking computing and personal resources                     |     build a   minimalistic solution that can be run by a single developer on a Raspberry Pi   (C2)    |
|     PPC is difficult to teach and to experience   in the limited time frame of a typical lesson           |     provide a propaedeutic solution that works in a school or university   teaching setting (C3)      |
|     related to development (D)                                                                            |                                                                                                       |
|     PPC often appears as the core functionality,   so far as to even require to be a main routine         |     PPC should be a network-level concern and separated from high-level   concerns (D1)               |
|     introducing   PPC functionality to a business logic often requires a complete rework                  |     enable   piecewise introduction of PPC into an existing legacy business logic codebase   (D2)     |
|     PPC frameworks require a specific tech stack   and introduce a lot of dependencies                    |     the client-side and the core of the server-side should be free of any   dependencies (D3)         |
|     coding   for any particular PPC framework locks the developer to a specific language                  |     let   client-side developers freely choose their language or keep using the legacy   one (D4)     |
|     ** related   to security concerns (S) **                                                              |                                                                                                       |
|     some PPC   frameworks are developed by non-experts in cryptography and are unsafe                     |     do not   reinvent the wheel, make use of existing and proven PPC frameworks (S1)                  |
|     PPC involves sensitive data that should not   be visible to the front-end or the outside              |     enable topologies with data flow confined to trusted machines on the   backend (S2)               |
|     many PPC   use cases involve a third-party researcher who must be able to run analyses                |     enable   topologies with control flow coming from an external researcher (S3)                     |
|     every PPC calculation requires a complete   re-evaluation by data security officers                   |     separate topology, protocol, and function so they can be assessed   independently (S4)            |
|     ** related to deployment and operation (O) **                                                         |                                                                                                       |
|     PPC often requires all parties to agree on   the exact same tech stack and IT environment             |     enable joint computation between parties using different hardware and   software (O1)             |
|     without   a lot of experience, it is often unclear which PPC framework is best suited   for a task    |     frameworks   should be replaceable without the need to rewrite the business logic (O2)            |
|     universal PPC solutions often have enormous   overhead in terms of space and processing               |     provide “small and fast” solutions that cater to the most often   encountered tasks (O3)          |

# Results

# Conclusion
