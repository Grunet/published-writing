# Handling Concurrent Load During an AWS Outage: A Tradeoff To Consider

## Table of Contents

- [The Main Compute Primitives for AWS](#the-main-compute-primitives-for-aws)
- [Control Planes vs Data Planes](#control-planes-vs-data-planes)
    * [Static Stability](#static-stability)
- [How Each Primitive Handles Concurrency During a Control Plane Outage](#how-each-primitive-handles-concurrency-during-a-control-plane-outage)
    * [EC2](#ec2)
    * [Fargate](#fargate)
    * [Lambda](#lambda)
- [The Takeaway](#the-takeaway)
- [References](#references)

## The Main Compute Primitives for AWS

There are 3 compute primitives in AWS (Amazon Web Services) that almost all of its other compute offerings are built on top of

- Virtual Machines (i.e. EC2, Elastic Cloud Compute)
- Containers (e.g. Fargate for ECS, Elastic Container Service, or EKS, Elastic Kubernetes Service)
- Functions (i.e. Lambda)

Each comes with its own set of tradeoffs, but there is one subtle tradeoff that only manifests during certain AWS outages.

To understand that tradeoff, we first need to understand the concepts of “control planes” and “data planes” of AWS services.

## Control Planes vs Data Planes

Every AWS compute service is separated into 2 logical components, a control plane and a data plane.

The data plane is responsible for actually running the hardware and software powering the compute. Think of the physical server running a virtual machine, for example.

The control plane is responsible for making changes to the data plane. If you want to add a new virtual machine to the data plane, you have to make a request to the control plane for it to do so on your behalf.

### Static Stability

Services are designed this way in part to be more fault tolerant. If an outage occurs in the control plane, the data plane will continue working without issue. 

And in general, outages in control planes are more common than outages in data planes.

This leads to the concept of “static stability”, where as long as your workload doesn’t depend on control planes, it will remain stable during most AWS outages.

## How Each Primitive Handles Concurrency During a Control Plane Outage

But your existing workloads being stable during an AWS outage might not be enough. What if there’s a surge in load that they need to respond to? Will they be able to scale up to meet that demand?

Specifically, there’s the question of the maximum concurrency a workload can support during an AWS service control plane outage.

The answer to this question (perhaps surprisingly) depends on the compute primitive involved.

### EC2

In normal times, in the face of increased concurrency a workload can autoscale up to handle it (e.g. an ASG, Autoscaling Group, can bring up more virtual machines).

However, during an outage of the EC2 control plane, this isn’t possible (i.e. autoscaling requires requests to the control plane).

This means that during the outage, the maximum concurrency a workload can support is fixed and cannot be increased. Any requests exceeding this limit will fail.

### Fargate

Fargate behaves similarly to EC2, as starting new tasks requires a request to the ECS or EKS control plane.

So during a control plane outage, any requests exceeding the fixed maximum concurrency of the workload will fail.

### Lambda

Lambda is the odd duck out. 

In normal times, in the face of increased concurrency a workload can start up multiple new Lambda execution environments to handle the load.

But the subtlety here is that this behavior is part of the Lambda data plane, NOT the control plane.

This means that during an outage of the Lambda control plane, a workload can still handle essentially arbitrary concurrency of requests (only limited by your account’s quota on concurrent executions).

![The AWS lambda icon on top of a sports podium for awarding medals like you'd find in the Olympics. There is a 1 beneath the lambda icon, then a 2 to the left of it, and a 3 to the right of it. Nothing is on top of the 2nd or 3rd place spots, they are empty.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/yn5hpw73lvw3099mls3t.png)

## The Takeaway

If you need to handle arbitrary concurrency while the control plane of a service is impaired, Lambda provides the best tradeoff.

Fargate or EC2 (or any other more managed service built on top of them, e.g. Elastic Beanstalk) will not be able to meet the need.

## References

1. [AWS whitepaper on fault isolation boundaries](https://docs.aws.amazon.com/pdfs/whitepapers/latest/aws-fault-isolation-boundaries/aws-fault-isolation-boundaries.pdf) that defines “static stability” and references the lack of ability of EC2-based workloads to autoscale during control plane outages
2. [Lambda whitepaper that says scaling occurs at the level of the data plane](https://docs.aws.amazon.com/whitepapers/latest/security-overview-aws-lambda/lambda-executions.html)