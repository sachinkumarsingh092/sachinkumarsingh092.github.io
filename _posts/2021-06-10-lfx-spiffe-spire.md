---
title: Adding a new health subsytem in SPIFFE/SPIRE 
tags: [cloud-native, open-source]
style: fill
color: warning
description: Challenges and learnings while working on SPIRE.
---

![SPIRE](https://branding.cncf.io/img/projects/spire/horizontal/color/spire-horizontal-color.png "SPIRE")

tl;dr: In this post, I introduce the basic concepts around SPIFFE and SPIRE and also give a brief discription of the work done during the LFX mentorship on the new SPIRE health subsystem.

# Introduction

A common problem in cloud-based distributed systems is how to authenticate access between components of a system. The Secure Production Identity Framework For Everyone (SPIFFE) enables a service to securely obtain an identity from a trusted SPIFFE provider, giving the service a way to authenticate itself in a platform agnostic way.

The SPIFFE Runtime Environment (SPIRE) is a production-ready open source implementation of the SPIFFE speciﬁcation. The SPIRE project (as well as SPIFFE) is hosted by the Cloud Native Computing Foundation (CNCF).

# Background and Concepts

### The SPIFFE standards

The Secure Production Identity Framework For Everyone (or SPIFFE) is a set of open source standards for software identity.

- **SPIFFE ID**

A SPIFFE ID is a string that functions as the unique name for a service. It is modeled as a URI and is made up of several parts. For example, *spiffe://example.com/myservice* is a valid SPIFFE ID, where *spiffe://* is the **URI scheme**, *example.com* is the trust domain name, and *myservice* is the name of the woarkload.

- **Trust Domain**

Trust domains are used to manage administrative and security boundaries within and between organizations, and every SPIFFE ID has the name of its trust domain embedded in it.  A trust domain could represent an individual, organization, environment or department running their own independent SPIFFE infrastructure.

- **SPIFFE Verifiable Identity Document (SVID)**

The SPIFFE Veriﬁable Identity Document (SVID) is a cryptographically-veriﬁable identity document that is used to prove a service’s identity to a peer. An SVID is considered valid if it has been signed by an authority within the SPIFFE ID’s trust domain.two types of identity documents are deﬁned for use as an SVID by the SPIFFE speciﬁcations: **X.509** and **JWT**.

- **SPIFFE Workload API**

The SPIFFE Workload API is a local, non-networked API that workloads use to get their current identity documents, trust bundles, and related information. More details are given [here](https://spiffe.io/docs/latest/spiffe-about/spiffe-concepts/#spiffe-workload-api).

{% include elements/figure.html image="../assets/spiffe-workload-api.png" caption="Workload API (source [1])" %}

- **Trust Bundle**

A trust bundle is a collection of one or more certificate authority (CA) root certificates that the workload should consider trustworthy. Each SVID type has a speciﬁc way that it is represented in this bundle (e.g. for X509-SVID, CA certiﬁcates representing the public key(s) are included).

### SPIRE, the SPIFFE Runtime Environment

SPIRE is a production-ready implementation of the SPIFFE APIs that performs node and workload attestation in order to securely issue SVIDs to workloads, and verify the SVIDs of other workloads, based on a predefined set of conditions.

A SPIRE deployment is composed of a SPIRE Server and one or more SPIRE Agents. A server acts as a signing authority for identities issued to a set of workloads via agents.

{% include elements/figure.html image="https://spiffe.io/img/server_and_agent.png" caption="SPIRE server and agent (source [2])" %}

The detailed description and components of server and agent is given [here](https://spiffe.io/docs/latest/spire-about/spire-concepts/#all-about-the-server) and [here](https://spiffe.io/docs/latest/spire-about/spire-concepts/#all-about-the-agent) respectively.

The basic concepts of SPIRE are as follows:

- **Workload Registration**

Workload registration tells SPIRE how to identify the workload and which SPIFFE ID to give it. A registration entry maps an identity – in the form of a SPIFFE ID – to a set of properties known as selectors that the workload must possess in order to be issued a particular identity. During workload attestation, the agent uses these selector values to verify the workload’s identity.

- **Attestation**

Attestation is the process through which information about workloads and their environment is discovered and asserted.

There are two ﬂavors of attestation in SPIRE: *node* and *workload attestation*.

  - **Node Attestation**

  Node attestation occurs when an agent starts for the ﬁrst time. In node attestation, the agent contacts the SPIRE Server and enters into an exchange in which the server aims to positively identify the node the agent is running on and all its related selectors.

  To accomplish this, a platform-speciﬁc plugin is exercised in both the agent and the server.

  Below is a given example of node attestation of a node running in AWS.

  {% include elements/figure.html image="../assets/node-attestation.png" caption="Node Attestation (source [1])" %}

  1. The agent gathers proof of the node’s identity calling an AWS API.
  2. The agent sends this proof of identity to the server.
  3. The server validates proof of identity obtained in step 2 by calling out to the AWS API and then creates a SPIFFE ID for the agent.

  - **Workload Attestation**

  Workload attestation is the process of determining the workload identity that will result in an identity document being issued and delivered. The attestation occurs any time a workload calls and establishes a connection to the SPIFFE Workload API (on every RPC call a workload makes to the API), and the process from there on is driven by a set of plugins on the SPIRE Agent. 

  The agent use certain properties of the locally available authorities (such as the node’s OS kernel, or a local kubelet running on the same node) in order to determine the properties of the process calling the Workload API. Properties include the User ID (uid), Group ID (gid), filesystem path on a UNIX system and the namespace and service account that the process is running in Kubernetes.

  {% include elements/figure.html image="../assets/workload-attestation.png" caption="Workload Attestation (source [1])" %}
  1. A workload calls the Workload API to request an SVID.
  2. The agent interrogates the node’s kernel to get the attributes of the calling process.
  3. The agent gets the discovered selectors.
  4. The agent determines the workload’s identity by comparing discovered selectors to registration entries and returns the correct SVID to the workload.

- **Registration entries**

For SPIRE to issue workload identities, it must ﬁrst be taught about the workloads expected or allowed in its environment; what workloads are supposed to run where, what their SPIFFE IDs and general shape should be. SPIRE learns this information via registration entries, which are objects that are created and managed using SPIRE APIs that contain the aforementioned information.

Three core attributes of registration entries are Parent ID (where a particular workload should be running), SPIFFE ID (when we see this workload, what SPIFFE ID should we issue it), and Binding info (binds SPIFFE IDs to the nodes and workloads that they are meant to represent).

A registration entry can describe either a group of nodes or a workload.

  - **Node Entries**

  Registration entries that describe a node (or a group of nodes) use selectors generated by node attestation to assign a SPIFFE ID, which can be referenced later when registering workloads. Node entries have their Parent ID set to the SPIFFE ID of the SPIRE Server, as it is the server which is performing attestation and asserting that the node in question does indeed match the selectors deﬁned bythe entry.

  - **Workload Entries**

  Registration entries that describe a workload use selectors generated by workload attestation to assign a SPIFFE ID to workloads when a certain set of conditions are met. When the Parent ID and selectors conditions are met, the workload can receive a SPIFFE ID.


# Work Done

During the mentorship, I worked on implementing a new SPIRE health subsytem. SPIRE uses [go-health](https://github.com/InVisionApp/go-health) to make health calls to different subsytems and getting the response. For `/live` checks, a simple HTTP 200 response validated the liveness of the subsystem and for `/ready` checks, the server's ability to fetch a bundle as a health check for readiness of the server. 

The new implementation uses different liveness and readiness checks for each subsystem and accumulates them to determine the global readiness and liveness of server and agent. For example, for CA health checks, both readiness and liveness is determined by whetehr or not the X509 CA was successfully signed.

For server, we used CA, manager, and catalog datastore as the most prominent subsytems to determine healthchekcs for. This is just initial design and more subsytems might be included to better determine the healthchecks.

To enable the server healthchecks, add the following lines in `conf/server/server.conf`:

```
health_checks {
        listener_enabled = true
        bind_address = "localhost"
        bind_port = "8080"
        live_path = "/live"
        ready_path = "/ready"
}
```

Now we can ping (using cURL) the localhost at port 8080 to find the live and readiness checks result:

```
$ curl http://localhost:8080/live	# for liveness check
$ curl http://localhost:8080/ready	# for readiness check
```

# Work remaining

Now that the server healthchecks are done and released, the agent subsystems are being identified for the agent's health checks.

# Conclusion

LFX mentorship was a wonderful experience for me. I learned a lot of useful stuff around microservice security and authentication, used tools like delve, SPIRE and learned golang best practices. I will try my best to further contibute to SPIFFE/SPIRE. Thanks to [Evan Gilman](https://github.com/evan2645) and [Andrew Harding](https://github.com/azdagron) for all their help and guidance and making this mentroship a fun learning experience.

# References

1. [Solving the Bottom Turtle](https://spiffe.io/book/)
2. [SPIFFE/SPIRE Documentations](https://spiffe.io/docs/latest/spiffe-about/overview/)
