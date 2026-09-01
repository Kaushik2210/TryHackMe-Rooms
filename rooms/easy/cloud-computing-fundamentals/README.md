# Cloud Computing Fundamentals

**Platform:** TryHackMe · **Type:** Concept Guide (no personal run captured — see note below)
**Primary domain:** Cloud Computing

> **Note:** This is a concept guide covering the material this TryHackMe room teaches, written from
> public documentation and reference material. It is not a personal walkthrough — no session evidence
> (screenshots, command output, room-specific answers) is included. Code examples below are generic
> reference examples, not captures from a completed session.

## Overview

Cloud computing is a model for delivering on-demand computing resources — servers, storage,
databases, networking — over the internet, billed on usage rather than owned as fixed capital
infrastructure. The National Institute of Standards and Technology (NIST) formalized the definition
in Special Publication 800-145, which describes five essential characteristics, three service
models, and four deployment models. Fundamentals rooms in this space exist to give students a
vendor-neutral vocabulary (IaaS vs PaaS vs SaaS, public vs private cloud) before moving into
provider-specific rooms (AWS, Azure, GCP).

## Core Concepts

### NIST's Five Essential Characteristics

NIST SP 800-145 defines cloud computing by five traits a service must exhibit:

1. **On-demand self-service** — a consumer can provision compute or storage automatically, without
   requiring human interaction with the provider.
2. **Broad network access** — capabilities are available over the network through standard
   mechanisms usable by heterogeneous clients (laptops, phones, thin clients).
3. **Resource pooling** — the provider's resources are pooled to serve multiple consumers using a
   multi-tenant model, with different physical and virtual resources dynamically assigned according
   to demand.
4. **Rapid elasticity** — capabilities can be elastically provisioned and released, sometimes
   automatically, to scale rapidly outward and inward with demand.
5. **Measured service** — resource usage is monitored, controlled, and reported, providing
   transparency for both provider and consumer (this underpins pay-as-you-go billing).

### The Three Service Models

| Model | What the provider manages | What the customer manages | Example |
|---|---|---|---|
| **IaaS** (Infrastructure as a Service) | Physical hardware, virtualization, networking, storage | OS, runtime, middleware, applications, data | A virtual machine — e.g. an AWS EC2 instance or Azure Virtual Machine |
| **PaaS** (Platform as a Service) | Everything in IaaS, plus the OS and runtime environment | Application code and configuration | A managed application platform — e.g. AWS Elastic Beanstalk, Azure App Service, Heroku |
| **SaaS** (Software as a Service) | The entire stack, including the application itself | User data and account-level configuration | A finished application delivered over the web — e.g. Gmail, Salesforce, Microsoft 365 |

The pattern across the three models is a shifting boundary of responsibility: IaaS hands the customer
the most control (and the most operational burden), SaaS hands them the least of either. This
boundary is formalized in cloud providers' "shared responsibility model," discussed below.

### The Four Deployment Models

NIST SP 800-145 also defines deployment models describing *who* the infrastructure serves:

- **Public cloud** — infrastructure provisioned for open use by the general public, owned and
  operated by a business, academic, or government organization (or some combination), and existing
  on the premises of the cloud provider. Example: a standard AWS or Azure account.
- **Private cloud** — infrastructure provisioned for exclusive use by a single organization, whether
  managed internally or by a third party, and whether hosted on- or off-premises.
- **Community cloud** — infrastructure provisioned for exclusive use by a specific community of
  consumers from organizations that share concerns (e.g. mission, security requirements, policy) —
  common in government or regulated-industry consortia.
- **Hybrid cloud** — a composition of two or more distinct infrastructures (private, community, or
  public) that remain unique entities but are bound together by standardized or proprietary
  technology enabling data and application portability (e.g. cloud bursting for load balancing).

### A Minimal Illustration: Interacting with Cloud APIs

Most cloud interaction in practice happens through a provider SDK or REST API rather than a web
console. A generic illustrative example using Python's `boto3` (the AWS SDK) shows the shape of
IaaS-level automation — listing storage buckets is a common first script:

```python
import boto3

s3 = boto3.client("s3")
response = s3.list_buckets()

for bucket in response["Buckets"]:
    print(bucket["Name"], bucket["CreationDate"])
# example-output-bucket 2024-01-15 09:32:11+00:00
# app-logs-prod         2023-11-02 14:05:47+00:00
```

This is illustrative, not a captured session — the point is that `boto3.client("s3")` authenticates
using credentials from the environment or a configured profile, and `list_buckets()` calls the
underlying S3 `ListBuckets` REST API. The same request/response pattern (SDK call -> signed HTTPS
request -> JSON or XML response) underlies almost every cloud automation and cloud security-scanning
tool.

## Why It Matters for Security

Cloud security is governed by the **shared responsibility model**: the provider is responsible for
security *of* the cloud (physical data centers, host infrastructure, hypervisor isolation), while the
customer is responsible for security *in* the cloud (identity and access management, data
encryption, network configuration, and — critically — how services are configured). This split moves
with the service model: in IaaS the customer manages far more (OS patching, firewall rules) than in
SaaS, where the provider manages nearly everything except account settings and data the customer
puts into the app.

The single most common real-world cloud breach pattern is a **misconfigured storage bucket** — an
AWS S3 bucket, Azure Blob Storage container, or Google Cloud Storage bucket left with public
read/write access when it was intended to be private. Because responsibility for bucket-level access
policy sits squarely with the customer under the shared responsibility model, this is a
customer-configuration failure, not a provider failure — and it has caused numerous large-scale data
exposures of customer records, credentials, and internal documents across affected organizations over
the years. Other recurring cloud-security failure modes include overly permissive IAM roles/policies
(granting `*:*` instead of least-privilege scoped permissions), exposed management consoles or APIs
without multi-factor authentication, hardcoded credentials in code pushed to public repositories, and
unpatched or unmonitored PaaS/IaaS workloads.

## Common Pitfalls / Misconfigurations

- **Public storage buckets/containers** by default or by mistake — always verify bucket policies and
  access-control lists explicitly deny public access unless specifically required.
- **Overly broad IAM policies** — granting wildcard permissions (`"Action": "*"`, `"Resource": "*"`)
  instead of scoping to the specific actions and resources a role needs.
- **Long-lived static credentials** (access keys checked into source control or left unrotated)
  instead of short-lived, role-based credentials.
- **No encryption at rest or in transit** for sensitive data stored in cloud services, despite most
  providers offering it as a low-friction, often default-available option.
- **Confusing "responsibility" with "no responsibility."** Even under SaaS, the customer is still
  responsible for who has access to their account and what data they put into it — the provider
  cannot secure a weak or reused password on the customer's behalf.
- **Treating hybrid/multi-cloud environments as uniformly configured** — security controls (logging,
  IAM, network segmentation) that are default-on in one provider or deployment model are often
  opt-in, or configured differently, in another.

## Related TryHackMe Rooms in This Series

This room is typically foundational for later provider-specific rooms (AWS, Azure, GCP) in the same
learning path. See `../python-simple-demo/README.md` for the scripting fundamentals that cloud
automation and cloud security-scanning tooling (including SDKs like `boto3`) build on.

## References

- NIST SP 800-145, *The NIST Definition of Cloud Computing*: https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-145.pdf
- NIST SP 800-145 landing page (CSRC): https://csrc.nist.gov/pubs/sp/800/145/final
- AWS, "Shared Responsibility Model": https://aws.amazon.com/compliance/shared-responsibility-model/
- Microsoft Azure, "Shared responsibility in the cloud": https://learn.microsoft.com/en-us/azure/security/fundamentals/shared-responsibility
- AWS `boto3` documentation (S3 client): https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html
- OWASP Cloud Security: https://owasp.org/www-project-cloud-security/
