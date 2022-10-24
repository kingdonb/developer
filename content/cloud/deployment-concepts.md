title = "Deployment Concepts"
template = "cloud_main"
date = "2022-03-14T00:22:56Z"
enable_shortcodes = true

---

* [Deployments in Fermyon Cloud](#deployments-in-fermyon-cloud)
* [Bindle - An Aggregate Object Storage System](#bindle---an-aggregate-object-storage-system)
* [The Deployment Process Explained](#the-deployment-process-explained)
  * [1. Packaging and Uploading an Application](#1-packaging-and-uploading-an-application)
  * [2. Create or Upgrade an Application](#2-create-or-upgrade-an-application)

## Deployments in Fermyon Cloud

Deploying applications to a cloud service should be simple. Even though there is complexity involved in operating a cloud with many servers, many applications, and an ever-changing number of workloads, the user's responsibility when deploying their applications should be minimal.

In this article, we describe the core technologies and concepts, which are part of the deployment process in the Fermyon Cloud.

## Bindle - An Aggregate Object Storage System

The Fermyon Cloud uses [Bindle](https://github.com/deislabs/bindle) to package and distribute Spin applications. Bindle is an open-source project, built and maintained by Deis Labs. Bindle is very well documented, so we will not go into details of how Bindle works, other than calling out a few core features of the system here:

- A bindle refers to the combined package, which always contains:
  - An invoice: A file with the bindles name, description, and a list of parcel manifests.
  - One or more parcels: The individual data components in the package.
- Bindles are immutable and cannot be overwritten.
  - Once a bindle is put into a bindle hub (the server), it cannot be changed, nor can it be deleted.
- Bindles can use semantic versioning [SemVer](https://semver.org) as part of their names.

## The Deployment Process Explained

In the Fermyon Cloud, we host an instance of Bindle, so when you run `spin deploy`, the command will take care of:
1. Packaging the application and upload it to the Fermyon Cloud
2. Creating or upgrade an application, using the bindle

There is no direct interaction with Bindle when using the Fermyon Cloud.

Let's unfold each of these steps.

### 1. Packaging and Uploading an Application

The first step in deploying an application is to package all the files into parcels and generate an invoice.  All of this is handled in a staging directory, which is either a temporary directory using [this implementation](https://doc.rust-lang.org/std/env/fn.temp_dir.html#platform-specific-behavior) or the staging directory defined using the `--staging-dir` option.

The bindle will be named using the `name` from `spin.toml`, the `version` from `spin.toml`, and a build metadata string, which is automatically generated by Spin at deployment time.

Bindles in the Fermyon Cloud always use Semantic Versioning and require major, minor, and patch version numbers to be specified. Therefore, the version in `spin.toml` has to conform to the `MAJOR.MINOR.PATCH` format, i.e. `1.0.1` is valid, `1.0` and `1` are not valid version numbers for a Spin application. The result is that the version of a Spin application will be like this `my_app_name/1.0.0+r80e5abb`.

Following packaging, the bindle will be uploaded to the Fermyon Cloud. As soon as a bindle has been uploaded, it cannot be modified or deleted. This is to preserve the integrity of the immutability of bindles.

### 2. Create or Upgrade an Application

The next step in the deployment process is to create or upgrade the application.

An application in the Fermyon Cloud can have multiple revisions, which are tied to channels. These concepts are derived from [Hippo](https://docs.hippofactory.dev/) an open-source Platform as a Service (PaaS) for WebAssembly. As you deploy your application both an application, a channel and a revision will be created in the Fermyon Cloud.

If the application already exists, an upgrade will take place. What happens at this point is that a new revision will be created, and as soon as this is deemed healthy, traffic will start to route to the new revision. The failover from the old to the new revision takes a short amount of time, during which you will be able to observe replies from both revisions. The application existence is determined based on the combination of the user account and the Spin application name, as defined in `spin.toml`.

The deployment process checks for the application health endpoint and finishes once the application is concluded to be healthy by the cloud. The application health point is an integral part of the Fermyon Cloud but does reserve the HTTP route `/.well-known/spin/health`, which will not be routed to your Spin application.