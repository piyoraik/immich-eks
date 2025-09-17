# EKS on Immich with Argo CD

This repository contains Kubernetes manifests to deploy [Immich](https://immich.app/) on Amazon EKS (Elastic Kubernetes Service) using Argo CD.

## Overview

This setup consists of the following components:

- **Immich**: The main Immich application, deployed via its official Helm chart.
- **PostgreSQL**: The database for Immich, deployed using the `app-template` Helm chart.
- **Redis**: The cache for Immich, deployed using the Bitnami Redis Helm chart.
- **Storage**:
  - A default `StorageClass` is created using the AWS EBS CSI Driver.
  - A `PersistentVolumeClaim` is created for Immich's library data.

All applications are managed as Argo CD `Application` resources for GitOps-style deployment.

## Prerequisites

- An existing EKS cluster.
- [Argo CD](https://argo-cd.readthedocs.io/en/stable/) installed in the cluster.
- An EKS cluster with the [AWS EBS CSI Driver](https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html) add-on enabled. This setup assumes the cluster is using EKS Auto Mode for the EBS CSI driver.

## Deployment Steps

1.  **Create Namespace and Storage Resources:**

    Apply the namespace, `StorageClass`, and `PersistentVolumeClaim` manifests. This will create the `immich` namespace and set up the necessary storage.

    ```bash
    kubectl apply -f namespace.yaml
    kubectl apply -f storageClass.yaml
    kubectl apply -f immichStorage.yaml
    ```

2.  **Deploy Applications with Argo CD:**

    Apply the Argo CD `Application` manifests. This will trigger Argo CD to deploy Immich, PostgreSQL, and Redis into the `immich` namespace.

    ```bash
    kubectl apply -f argocd/
    ```

    Argo CD will then automatically sync the applications based on the manifests in this repository.

## File Descriptions

- `namespace.yaml`: Creates the `immich` namespace where all resources will be deployed.
- `storageClass.yaml`: Defines a default `StorageClass` named `auto-ebs-sc` that uses the `ebs.csi.eks.amazonaws.com` provisioner. This is intended for use with the EKS Auto Mode for the EBS CSI driver, which simplifies dynamic volume provisioning.
- `immichStorage.yaml`: Creates a `PersistentVolumeClaim` named `immich-library` for storing Immich's photo and video library.
- `argocd/`: This directory contains the Argo CD `Application` manifests.
  - `immich.yaml`: Deploys the Immich application using the official Helm chart. It is configured to use the external PostgreSQL and Redis deployments.
  - `postgres.yaml`: Deploys a PostgreSQL database for Immich using the `app-template` Helm chart.
  - `redis.yaml`: Deploys a Redis instance for Immich using the Bitnami Helm chart.

## License

This project is licensed under the Apache License 2.0. See the [LICENSE](LICENSE) file for details.
