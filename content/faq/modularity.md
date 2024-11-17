---
title: "Do I need to use all the features?"
linkTitle: "Modularity"
weight: 3
description: >
  Understanding the modular structure of fireact.dev
---

No, fireact.dev is built with modularity in mind. The framework consists of three main packages:

* @fireact.dev/core - Essential authentication and user management
* @fireact.dev/saas - Subscription management and billing
* @fireact.dev/saas-cloud-functions - Backend processing and Stripe integration

You can start with just the core package for basic authentication features and add other packages as needed. Note that if you want to use the SaaS features, you'll need both @fireact.dev/saas and @fireact.dev/saas-cloud-functions packages together.
