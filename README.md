# csi-release

The Kubernetes
[Storage SIG](https://github.com/kubernetes/community/tree/master/sig-storage)
maintains a set of components under the
[kubernetes-csi](https://github.com/kubernetes-csi) GitHub
organization.

This repository contains code and documentation for building a
kubernetes-csi release.


Key concepts
============

Versioning
----------

Each of these components has its own versioning and release
notes. [Semantic versioning](https://semver.org/) is used.  Each
component runs its own unit testing before a release, resulting in a
tagged source code revision.

A "kubernetes-csi release" is a specific set of component releases
that were tested together. Each kubernetes-csi release has its own
version number which is increased according to the same rules as the
individual components (major version bump when any of its components
had a major change, etc.).

The `csi-release` repo references other components via their base
version and then picks up the latest release of each component with
that base version. The `csi-release` repo will once multiple multiple
different releases need to be supported, i.e. in contrast to
Kubernetes itself, the release scripts and their configuration can be
different for different release branches.

Release artifacts
-----------------

Primarily the content of each release are container images for the
different CSI sidecar apps. Eventually auxiliary files like the RBAC
rules that are included in each component might also get published in
a single location.

Only binaries provided as part of such a release should be considered
production ready. Binaries are never going to be rebuilt, therefore an
image like `csi-node-driver-registrar:v1.0.2` will always pull the
same content and `imagePullPolicy: Always` can be omitted.

For those who want to allow automatic updates to more recent releases
without explicitly updating a deployment, each image will also be
published with the last and the last two numbers stripped, i.e. as
`csi-node-driver-registrar:v1.0` and
`csi-node-driver-registrar:v1`. Because those images will change over
time, `imagePullPolicy: Always` is needed.

Release process
---------------

* Maintainers tag individual components.

* A single
  [Prow](https://github.com/kubernetes/test-infra/blob/master/prow/README.md)
  job checks out the source of all components and builds them.

* Container images are placed in a staging area specific to the current build.
  Staging areas from previous builds that are older than a certain amount
  of time are garbage collected to free up the space.

* The job runs component unit tests (aka `make test` in each component
  repo), sanity and E2E tests (from
  [csi-test](https://github.com/kubernetes-csi/csi-test), with the
  definition of what to test in which configurations stored in this
  repo).

* After reviewing results and potentially some more manual tests,
  someone must trigger the promotion of release artifacts from the
  staging area to the production area and announce the release.


Implementation
==============

* One periodic prow job which brings up a test cluster. TODO (?): try with different clusters?

* Container images published under k8s.gcr.io. TODO: namespace for kubernetes-csi?

* Each staging area is identified with a UUID. Stagging images get that UUID as tag.

* Other artifacts under `https://dl.k8s.io/kubernetes-csi/[staging UUID|release number]`.

