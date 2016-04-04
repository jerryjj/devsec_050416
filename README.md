# Secrets in Kubernetes

This is a presentation I held at "DevOps and Security" -meetup on 5th of April 2016.

## Presentation

In PDF format available here Kubernetes+Secrets.pdf

## Demos

These demos expect that you have a Kubernetes cluster running
and your environment has been configured for it. (Credentials set and SDK installed)

* [Secrets basics](https://github.com/jerryjj/devsec_050416/blob/master/demo/secrets.md)

### My setup

I used Google Container Engine (GKE) for my setup.
Here are the steps I used to create my cluster

```sh
export PROJECT_ID=[my-project-id]
gcloud config set project $PROJECT_ID
gcloud container clusters create devsec --num-nodes 2 --zone europe-west1-b
export CLUSTER_CONTEXT=$(kubectl config view | awk '{print $2}' | grep "devsec" | tail -n 1)
```

Then when ever I call the _kubectl_ -command I actually call:

```sh
kubectl --context=$CLUSTER_CONTEXT ...
```

This is because I control multiple clusters in different projects and
this makes sure I always run the commands in the right cluster...
