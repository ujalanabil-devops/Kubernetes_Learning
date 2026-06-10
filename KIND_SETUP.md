# KIND CLUSTER SETUP STEP BY STEP:-
Kind is a tool for running local Kubernetes clusters using Docker container "nodes". kind was primarily designed for testing Kubernetes itself, but may be used for local development or CI.

kind consists of:

1. Go packages implementing cluster creation, image build, etc.
2. A command line interface (kind) built on these packages.
3. Docker image(s) written to run systemd, Kubernetes, etc.
4. kubetest integration also built on these packages (WIP)
5. kind bootstraps each "node" with kubeadm. 

For more details see the design documentation.
https://kind.sigs.k8s.io/docs/design/initial 

