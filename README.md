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

# Results

# Conclusion
