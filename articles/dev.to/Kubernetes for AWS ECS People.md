# Kubernetes for AWS ECS People

## Background

I've been trying to get familiar with key Kubernetes (k8s) terms recently by going through this very helpful [Understanding Kubernetes in a visual way](https://www.amazon.com/Understanding-Kubernetes-visual-way-sketchnotes/dp/B0BB619188) book, and noticed a lot of analogies to AWS's Elastic Container Service (ECS) offering while doing so.

Since I'm more familiar with ECS terms, I thought I'd take a stab at drafting a "translation guide" of sorts between k8s and ECS terminology. The disclaimers being I currently know very little about k8s and am no AWS expert.

## The Translation Guide

This is in the same order as the terms appear in the [Understanding Kubernetes in a visual way](https://www.amazon.com/Understanding-Kubernetes-visual-way-sketchnotes/dp/B0BB619188) book

<dl>
    <dt>etcd</dt>
    <dd>ECS Control Plane's state tracking system</dd>
    <dt>API server</dt>
    <dd>ECS Control Plane's implementation behind the public ECS API surface</dd>
    <dt>Scheduler</dt>
    <dd>ECS Control Plane's task placement functionality</dd>
    <dt>Controller-Manager</dt>
    <dd>ECS Control Plane's various other functionalities that try to make the current state of things match the desired state</dd>
    <dt>Kubelet</dt>
    <dd>ECS agent on the EC2 instance</dd>
    <dt>Proxy</dt>
    <dd>Elastic Network Interface (ENI) for the EC2 instance</dd>
    <dt>Kubeconfig</dt>
    <dd>Cloudformation template</dd>
    <dt>Kubectl</dt>
    <dd>AWS (ECS) CLI</dd>
    <dt>Namespaces</dt>
    <dd>ECS Clusters</dd>
    <dt>Resource Quotas</dt>
    <dd>ECS service quotas</dd>
    <dt>Pod</dt>
    <dd>Task</dd>
    <dt>Pod Lifecycle (States)</dt>
    <dd>Task Lifecycle (States)</dd>
    <dt>Pod Deletion</dt>
    <dd>ECS stopping a broken task then spinning up a fresh one</dd>
    <dt>Liveness probe</dt>
    <dd>Container-level healthcheck</dd>
    <dt>Readiness probe</dt>
    <dd>ALB healthchecks</dd>
    <dt>Startup probe</dt>
    <dd>ECS service health check grace period</dd>
    <dt>"PodStart" container lifecycle event</dt>
    <dd>Task Definition command (?)</dd>
    <dt>"PreStop" container lifecycle event</dt>
    <dd>Container code listening for SIGTERM sent by the ECS agent (?)</dd>
    <dt>Execute commands in containers</dt>
    <dd>ECS Exec</dd>
    <dt>Init containers</dt>
    <dd>Task Definition container dependency settings</dd>
    <dt>Job</dt>
    <dd>EventBridge event triggering a Task execution</dd>
    <dt>CronJob</dt>
    <dd>Scheduled Tasks (aka EventBridge rule with a cron schedule triggering a Task execution)</dd>
    <dt>ConfigMaps</dt>
    <dd>Task Definition environment variables</dd>
    <dt>Secrets</dt>
    <dd>Using Parameter Store/Secrets Manager secrets in a Task Definition</dd>
    <dt>Deployment Rolling Update</dt>
    <dd>The default ECS service update/deploy behavior</dd>
    <dt>Pull images configuration</dt>
    <dd>ECS agent pulls images from ECR per the Task Definition</dd>
    <dt>ReplicaSet</dt>
    <dd>DesiredCount for an ECS service</dd>
    <dt>DaemonSet</dt>
    <dd>DAEMON scheduling strategy for an ECS service</dd>
    <dt>Service</dt>
    <dd>Network Load Balancer</dd>
    <dt>Labels and Selectors</dt>
    <dd>Tags and the "Resource Groups Tagging" API</dd>
    <dt>Ingress</dt>
    <dd>Application Load Balancer (ALB)</dd>
    <dd>API Gateway</dd>
    <dt>Persistent Volume, Persistent Volume Claim, StorageClass</dt>
    <dd>Elastic Block Storage (EBS)</dd>
    <dd>Elastic File System (EFS)</dd>
    <dt>Horizontal Pod Autoscaler</dt>
    <dd>Application Auto Scaling rules applied to an ECS service</dd>
    <dt>Limit Range</dt>
    <dd>? (are there ways to limit memory usage at the ECS cluster level?)</dd>
    <dt>Out of Memory (OOM) Killer</dt>
    <dd>ECS agent on the EC2 instance killing a task if it runs out of memory</dd>
    <dt>Memory Requests and Limits</dt>
    <dd>Container-level memoryReservation and memory parameters in a Task Definition</dd>
    <dt>CPU Requests and Limits</dt>
    <dd>Container-level cpu parameter in a Task Definition</dd>
    <dt>Pod Disruption Budget</dt>
    <dd>Minimum Healthy Percent and Maximum Healthy Percent for an ECS service during a deployment</dd>
    <dt>Quality of Service</dt>
    <dd>?</dd>
    <dt>Network Policies</dt>
    <dd>Network Access Control Lists (ACLs)</dd>
    <dt>Role-Based Access Control (RBAC)</dt>
    <dd>IAM policies and Resource-based policies</dd>
    <dt>Pod (Anti) Affinity and Node (Anti) Affinity</dt>
    <dd>ECS service task placement strategies and task placement constraints</dd>
    <dt>Node</dt>
    <dd>EC2 instance</dd>
    <dt>Node lifecycle</dt>
    <dd>EC2 instance lifecycle</dd>
</dl>

## Request for Help

If you are more familiar with k8s or ECS and notice any inaccuracies or analogies that could be improved (there are probably a bunch!), feel free to comment here and I can edit them in.
    
