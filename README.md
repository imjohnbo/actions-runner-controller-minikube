# Actions Runner Controller Minikube

This repo is for testing out [actions-runner-controller](https://github.com/summerwind/actions-runner-controller) on Minikube.

## Setup

- Set up `minikube` ([instructions](https://minikube.sigs.k8s.io/docs/start/)). I made the Docker driver default: `minikube config set driver docker`.
- `minikube start`
- Set up cert-manager using [these instructions](https://cert-manager.io/docs/installation/kubernetes/).
- Adjust `--sync-period=` in [actions-runner-controller.yaml](actions-runner-controller.yaml). If the default of 10m is ok for you, you could instead use the [source spec](https://github.com/summerwind/actions-runner-controller/releases/download/v0.12.0/actions-runner-controller.yaml).
- Install it in the cluster: `kubectl apply -f actions-runner-controller.yaml` (or `kubectl apply -f https://github.com/summerwind/actions-runner-controller/releases/download/v0.12.0/actions-runner-controller.yaml`).
- [Create a new GitHub App](https://github.com/summerwind/actions-runner-controller#using-github-app), set its environment variables, then create a Kubernetes secret:
```
kubectl create secret generic controller-manager \
    -n actions-runner-system \
    --from-literal=github_app_id=${APP_ID} \
    --from-literal=github_app_installation_id=${INSTALLATION_ID} \
    --from-file=github_app_private_key=${PRIVATE_KEY_FILE_PATH}
```

## Usage
- Create a Runner or RunnerDeployment spec and apply it to the cluster, e.g. `kubectl apply -f runnerdeployment.yaml`. Find more examples [here](https://github.com/summerwind/actions-runner-controller#usage). 🎉
- Autoscaling guidance [here](https://github.com/summerwind/actions-runner-controller#autoscaling). The controller checks the GitHub API for Queued and In Progress job every `--sync-period` minutes, matching it to the number of available runners and making a new runner available as necessary.
- Note you can also use [custom labels](https://docs.github.com/en/free-pro-team@latest/actions/hosting-your-own-runners/using-labels-with-self-hosted-runners) like `my-runner` to give more specificity to your workflows.

```yaml
...

jobs:
  build:
    runs-on: [self-hosted, my-runner]
    steps:
    - ...
```

![runner](https://user-images.githubusercontent.com/2993937/99438831-5a70e480-28e2-11eb-9834-cb843d0d9e43.png)

## Credits
https://github.com/summerwind/actions-runner-controller, licensed under [Apache License Version 2.0](LICENSE).
