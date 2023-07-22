# lab2-cloud-service-availability
In this Lab, we will deploy a set of monitoring and alert services, then optimize the service through the high availability method we learned

## Introduction

> What is availability?

> How to measure availability?

> What are the common factors that affect the availability of services, and how to solve these problems.

## Learning Objectives

- Learn the definition and measurement of availability
- Learn some best practices about high availability

## Pre-requisites
Lab2 has the same prerequisites as lab1. it will speed up the progress of lab2 if you have completed lab1.

- An AWS account
- [`kubectl`](https://kubernetes.io/docs/tasks/tools/install-kubectl/): the standard Kubernetes command line interface
- Node.js 16. Use nvm to [install Node.js 16](https://github.com/nvm-sh/nvm#installing-and-updating):

  ```bash
  $ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.3/install.sh | bash
  $ nvm install 16
  ```

- Basic knowledge of AWS, Kubernetes, RDBMS
- VPN for connecting to AWS API and GitHub

## Syllabus

1. Create a new GitHub repository base on [this template](https://github.com/vldbss-2023/lab2-cloud-service-availability)
   * Clone the newly created repository
   * In the root directory of the repository, run `make install` to install the dependencies. 
2. (30 min) Create an EKS cluster [`1-create-an-eks-cluster`](./1-create-an-eks-cluster/README.md)
3. (20 min) Deploy observability service
   Operator [`2-deploy-observability-service`](./2-deploy-observability-service/README.md)
4. (50 min) Solve availability-related issues[`3-solve-availability-related-issues`](./3-solve-availability-related-issues/README.md)
5. (20 min) Optimize observability service
   Operator [`4-optimize-observability-service`](./4-optimize-observability-service/README.md)
6. Bonus: Deploy a TiDB cluster and monitor it. 

---

## AWS billing price

This lab will incur charges under the aws account, described in detail at: 

- New EKS cluster control plane, **_1_** cluster x **_0.10_** USD per hour
- Two EKS worker EC2 `t2.medium` instances, **_2_** instances * **_0.0464_** USD per hour
- 4 EBS of size 1 GiB, with negligible cost

Total **_0.1928_** USD per hour.

## Criteria for judging
The fourth step of our lab will be hidden first, and students' thinking about the problems in the third step will be the focus of judgment
1. Complete the deployment operations in 1-create-an-eks-cluster and 2-deploy-observability-service. (0-20)
2. For each question in 3-solve-availability-related-issues, 0-10 points can be obtained if only the text description is provided, and 0-15 points can be obtained if the code is directly modified.
3.Answers to the questions are given in 4-operation and the practical work is done. (0-20)
