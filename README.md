# Microsoft Azure: Hands-on AKS workshop </br> Implementing zero-trust security for containers

## Welcome

In this AKS-focused workshop, you will work with Microsoft Azure and Calico Cloud to learn how implement zero-trust security for workloads to reduce the attack surface of applications running on AKS.  

Cloud-native applications require a modern approach based on the zero-trust principles of identity-based access, least privilege access, and proactively detecting threats and reducing the blast radius in case of a breach.

Calico Cloud enables fine-grained, zero-trust workload access controls between your microservices and external databases, cloud services, APIs, and other applications. It also prevents the lateral movement of threats with identity-aware segmentation that works across all of your workload environments, including hosts, VMs, Kubernetes components, and services.

You will come away from this workshop with an understanding of how others in your industry are securing and observing cloud-native applications in Microsoft Azure, along with best practices that you can implement in your organization.

### Time Requirements

The estimated time to complete this workshop is 60-90 minutes.

### Target Audience

- Cloud Professionals
- DevSecOps Professional
- Site Reliability Engineers (SRE)
- Solutions Architects
- Anyone interested in Calico Cloud :)

### Learning Objectives

1. Learn how to deploy zero-trust workload access controls
2. Extend firewall protection at the granular, workload level
3. Block lateral movement of APTs with identity-aware microsegmentation
4. Understand how to apply zero-trust security controls at application level.

## Modules

This workshop is organized in sequential modules. One module will build up on top of the previous module, so please, follow the order as proposed below.
 
Module 1 - [Getting Started](/modules/module-2-getting-started.md)  
Module 2 - [Deploy an Azure AKS cluster](/modules/module-2-deploy-aks.md)  
Module 3 - [Connect the AWS EKS cluster to Calico Cloud](/modules/module-4-connect-calicocloud.md)  
Module 4 - [Create the test environment](/modules/module-5-test-environment.md)  
Module 5 - [Enable egress gateway support](/modules/module-6-egw-support.md)  
Module 6 - [Deploy Egress Gateway and use a pod selector](/modules/module-7-egw-perpod.md)  
Module 7 - [Deploy Egress Gateway and use a namespace selector](/modules/module-8-egw-pernamespace.md)  
Module 8 - [Deploy Egress Gateway with an AWS elastic IP](/modules/module-9-egw-elastic-ip.md)  
Module 9 - [Clean up](/modules/module-10-clean-up.md)  

> **Note**: The examples and sample code provided in this workshop are intended to be consumed as instructional content. These will help you understand how Calico Cloud can be configured to build a functional solution. These examples are not intended for use in production environments.