# kubernetes-gitlab
Manifests to deploy GitLab on Kubernetes 

Installation process described in [blog](http://blog.lwolf.org/post/how-to-easily-deploy-gitlab-on-kubernetes/)

# Helm chart based on this manifests
[https://github.com/lwolf/gitlab-chart](https://github.com/lwolf/gitlab-chart)


## 2016-11-24 Update #1 after post was published

### Bump versions

* gitlab-runner:v1.8.0
* gitlab:8.16.3
* postgresql:9.5-3
* redis:3.2.4 (official redis container)

### Change in ingress

* SSH is now available through 1022 service post.
* NGINX settings is now configurable with configmap nginx-settings-configmap.yml.
 Which currently sets body-size to 0 and increases timeouts to avoid timeouts. 


# TL;DR

## Deploying GitLab itself
```
# create gitlab namespace
> $ kubectl create -f gitlab-ns.yml

# deploy redis
> $ kubectl create -f gitlab/redis-svc.yml
> $ kubectl create -f gitlab/redis-deployment.yml

# deploy postgres
> $ kubectl create -f gitlab/postgresql-svc.yml
> $ kubectl create -f gitlab/postgresql-deployment.yml

# deploy gitlab itself
> $ kubectl create -f gitlab/gitlab-svc.yml
> $ kubectl create -f gitlab/gitlab-svc-nodeport.yml
> $ kubectl create -f gitlab/gitlab-deployment.yml

# deploy ingress controller
> $ kubectl create -f ingress/default-backend-svc.yml
> $ kubectl create -f ingress/default-backend-deployment.yml
> $ kubectl create -f ingress/nginx-settings-configmap.yml
> $ kubectl create -f ingress/configmap.yml
> $ kubectl create -f ingress/nginx-ingress-lb.yml
> $ kubectl create -f ingress/gitlab-ingress.yml

```

## Deploying GitLab Runner

```
# deploy Minio
kubectl create -f minio/minio-svc.yml
kubectl create -f minio/minio-deployment.yml

# check that it's running
kubectl get pods --namespace=gitlab

    gitlab   minio-2719559383-y9hl2    1/1    Running   0   1m

# check logs for AccessKey and SecretKey
> $ kubectl logs -f minio-2719559383-y9hl2 --namespace=gitlab
+ exec app server /export

Endpoint:  http://10.2.23.7:9000  http://127.0.0.1:9000
AccessKey: 9HRGG3EK2DD0GP2HBB53
SecretKey: zr+tKa6fS4/3PhYKWekJs65tRh8pbVl07cQlJfkW
Region:    us-east-1

Browser Access:
   http://10.2.23.7:9000  http://127.0.0.1:9000

Command-line Access: https://docs.minio.io/docs/minio-client-quickstart-guide
   $ mc config host add myminio http://10.2.23.7:9000 9HRGG3EK2DD0GP2HBB53 zr+tKa6fS4/3PhYKWekJs65tRh8pbVl07cQlJfkW

Object API (Amazon S3 compatible):
   Go:         https://docs.minio.io/docs/golang-client-quickstart-guide
   Java:       https://docs.minio.io/docs/java-client-quickstart-guide
   Python:     https://docs.minio.io/docs/python-client-quickstart-guide
   JavaScript: https://docs.minio.io/docs/javascript-client-quickstart-guide


# create bucket named `runner`
> $ kubectl exec -it minio-2719559383-y9hl2 --namespace=gitlab -- bash -c 'mkdir /export/runner'

# register runner with token found on http://yourrunning-gitlab/admin/runners
> $ kubectl run -it runner-registrator --image=gitlab/gitlab-runner:v1.5.2 --restart=Never -- register
Waiting for pod default/runner-registrator-1573168835-tmlom to be running, status is Pending, pod ready: false

Hit enter for command prompt

Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/ci):
http://gitlab.gitlab/ci
Please enter the gitlab-ci token for this runner:
_TBBy-zRLk7ac1jZfnPu
Please enter the gitlab-ci description for this runner:

Please enter the gitlab-ci tags for this runner (comma separated):
shared,specific
Registering runner... succeeded                     runner=_TBBy-zR
Please enter the executor: virtualbox, docker+machine, docker-ssh+machine, docker, docker-ssh, parallels, shell, ssh:
docker
Please enter the default Docker image (eg. ruby:2.1):
python:3.5.1
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
Session ended, resume using 'kubectl attach runner-registrator-1573168835-tmlom -c runner-registrator -i -t' command when the pod is running

# delete temporary registrator
> $ kubectl delete deployment/runner-registrator


```

Edit `gitlab-runner/gitlab-runner-docker-configmap.yml` and fill in your runner token and minio credentials.

```
# deploy configmap and runnner itself
> $ kubectl create -f gitlab-runner/gitlab-runner-docker-configmap.yml
> $ kubectl create -f gitlab-runner/gitlab-runner-docker-deployment.yml
```
