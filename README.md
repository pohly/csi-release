# csi-release

The Kubernetes
[Storage SIG](https://github.com/kubernetes/community/tree/master/sig-storage)
maintains a set of components under the
[kubernetes-csi](https://github.com/kubernetes-csi) GitHub
organization.

Key concepts
============

Versioning
----------

Each of these components has its own versioning and release
notes. [Semantic versioning](https://semver.org/) is used.

A "kubernetes-csi release" is a specific set of component releases
that were tested together. It's called a "combined release". It's not
required that exactly those components are used together. In
particular, individual components might also get minor updates after a
kubernetes-csi release without updating that combined release.

The
[hostpath example deployment](https://github.com/kubernetes-csi/csi-driver-host-path/tree/master/deploy)
defines the components that are part of a combined release. This
implies that the hostpath driver repo must be updated and tagged to
create a new combined release. Therefore the hostpath driver's version
becomes the kubernetes-csi release version, which is increased
according to the same rules as the individual components (major
version bump when any of its components had a major change, etc.).

Release artifacts
-----------------

Tagging a component with a semantic version number triggers a release
build for that component. The output is primarily the container image
for the component. Eventually auxiliary files like the RBAC rules that
are included in each component might also get published in a single
location.

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

* A change is submitted against master.

* A [Prow](https://github.com/kubernetes/test-infra/blob/master/prow/README.md)
  job checks out the source of a modified component, rebuilds it and then
  runs unit tests and E2E tests with it.

* Maintainers accept changes into the master branch.

* The same Prow job runs for master and repeats the check. If it succeeds,
  a new "canary" image is published for the component.

* In preparation for the release of a major new update, a feature freeze is
  declared for the "master" branch and only changes relevant for that next
  release are accepted.

* When all changes targeted for the release are in master, automatic
  test results are okay and and potentially some more manual tests,
  maintainers tag each component directly on the master branch.

* Maintenance releases are prepared by branching a "release-X.Y" branch from
  release "vX.Y" and backporting relevant fixes from master. The same
  prow job as for master also handles the maintenance branches, but potentially
  with a different configuration.

Implementation
==============

Each component has its own release configuration (what to build and
publish) and rules (scripts, makefile). The advantage is that those
can be branched and tagged together with the component.

To simplify maintenance and ensure consistency, the common parts can
be shared via
[csi-release-tools](https://github.com/kubernetes-csi/csi-release-tools/).

The prow job then just provides a common execution environment, with
the ability to bring up test cluster(s), publish container images on
k8s.gcr.io (TODO: namespace for kubernetes-csi?) and other files under
`https://dl.k8s.io/kubernetes-csi/`.
