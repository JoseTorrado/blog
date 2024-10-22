+++
author = "Jose Torrado"
title = "Authoritative vs Non-Authoritative Properties in Terraform"
date = "2024-10-21"
description = "And when to use which"
tags = [
    "infrastructure",
    "cloud",
]
categories = [
    "Terraform",
    "IaC",
    "DevOps"
]
image = "cover.png"
+++

## First things First

I have not yet talked about infrastructure too much on this blog, but it is a big player in data engineering work. 

[Terraform](https://www.terraform.io/) is what I primarily use as a tool to manage cloud infrastructure as code - and it is very popular.

I should dedicate a full series on infrastructure and the significance it has in our field. In the mean time, this is an interesting [article](https://www.linkedin.com/pulse/data-engineering-101-three-types-engineers-you-need-know-matt-brady/).

## The actual intro 

Alright, now when using Terraform to manage IAM bindings, in GCP for example, it is **crucial** to understand auth vs non-auth properties. 

## Authoritative Properties 

`Authoritative Properties` mean that terraform has full control over the resource.

This means any changes made outside of terraform - like a manual adjustment in the cloud console - will be 
overwritten on the next `terraform apply`

You want to use these when you need strict compliance and consistency. For example, managing roles for a critical production environment. Any unauthorized changes outside of terraform can lead to security vulnerabilities.

The `google_project_iam_binding` resource is Authoritative in nature:

```hcl
resource "google_project_iam_binding" "editor" {
  project = "your-project-id"
  role    = "roles/editor"

  members = [
    "user:example-user@example.com",
  ]
}
```
## Non Authoritative Properties 

These will allow more flexibility. It allows terraform to manage aspects fo a resource without overwriting
changes made by other systems, tools or processes.

This comes in handy when you have multiple teams or systems managing a set of resources. Non-authoritative management allows teams to "coexist" without overwriting each other's changes

The `google_project_iam_member` resource is Non-Authoritative:

```hcl
resource "google_project_iam_member" "viewer" {
  project = "your-project-id"
  role    = "roles/viewer"
  member  = "user:another-user@example.com"
}
```

## Closing

As always, it depends. If you need strict compliance, you might want to opt for auth. If you need more flexibility or a loosely coupled system however, you might want to go for non-auth

What's important is to know what tools are available.
