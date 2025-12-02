## General Description:

I would like to deploy Django & Postgresql on k8s in automatic way with DevOps methodology.

# Links to external sources:

Deploy Django into Production with Kubernetes, Docker, & Github Actions. Complete Tutorial Series:
- https://www.youtube.com/watch?v=NAOsLaB6Lfc&t=14254s&ab_channel=CodingEntrepreneurs

How To Deploy a Scalable and Secure Django Application with Kubernetes:
- https://www.digitalocean.com/community/tutorials/how-to-deploy-a-scalable-and-secure-django-application-with-kubernetes 
- https://github.com/do-community/django-polls

Deploy a Django app with Kubernetes in 20 minutes:
- https://www.architect.io/blog/2022-08-04/deploy-python-django-kubernetes/ 

How to Deploy a Django Application With Kubernetes:
- https://betterprogramming.pub/how-to-deploy-a-django-application-with-kubernetes-f5814b0688bf 

How To Deploy A Django App Over A Kubernetes Cluster (With Video):
- https://medium.com/@tech_with_mike/how-to-deploy-a-django-app-over-a-kubernetes-cluster-with-video-bc5c807d80e2

How to orchestrate your Django application with Kubernetes:
- https://mattermost.com/blog/orchestrate-django-application-with-kubernetes/

Deploy a Scalable and Secure Django Application with Kubernetes:
- https://bobcares.com/blog/deploy-a-scalable-and-secure-django-application-with-kubernetes/

Django With CI/CD (Docker Container & Kubernetes):
- https://uzzal2k5.github.io/django-docker/

Some tips to deploy Django in kubernetes:
- https://www.jujens.eu/posts/en/2021/Mar/29/deploy-django-kubernetes/

Kubernetes Guide - Deploying a machine learning app built with Django, React and PostgreSQL using Kubernetes:
- https://datagraphi.com/blog/post/2021/2/10/kubernetes-guide-deploying-a-machine-learning-app-built-with-django-react-and-postgresql-using-kubernetes



## First Step: Initial Configuration & Fight with GitLab:
Start Based on: based on: https://www.youtube.com/watch?v=SlHBNXW1rTk&amp;list=PLEsfXFp6DpzRMby_cSoWTFw8zaMdTEXgL

```
alias k='kubectl'
sudo apt install python3-pip
touch requirements.txt
-create branch remotly on gitlab
export GIT_SSH_COMMAND="ssh -i ~/.ssh/id_rsa1"
git clone git@gitlab.internal.ericsson.com:emajprz/backup_and_draft.git
git add .
git commit -m "first commit"
git push -u origin main
git add .
git commit -m "initial setup"
git branch -M Django_on_k8s
git push -uf origin Django_on_k8s
touch .gitignore
```

## Create Python VIrtal Env:

```
python3.8 -V
kubectl version --client
python3.8 -m venv venv
source venv/bin/activate
pip freeze
pip install pip --upgrade
touch requirements.txt
pip install -r requirements.txt 

```

## DJango - Start Projects & enviroment variable handling:

```
cd web
django-admin startproject django_k8s .
touch .env 
- add code
python3.8 manage.py shell
import os
print(os.environ.get("REGION"))
```

```git add -all && git commit -m 'something' ```

## Start with Docker:

We need to have docker file to build application correctly from dir:
```
FROM python:3.8.10-slim

COPY . /app
WORKDIR /app

RUN python3 -m venv /opt/venv

RUN /opt/venv/bin/pip install pip --upgrade && \
    /opt/venv/bin/pip install -r requirements.txt && \
    chmod +x entrypoint.sh

CMD ["/app/entrypoint.sh"]
```
Then we need to have a way to run app (using sh script):
```
#!/bin/bash
APP_PORT=${PORT:-8000}
cd /app/
/opt/venv/bin/gunicorn --worker-tmp-dir /dev/shm django_k8s.wsgi:application --bind "0.0.0.0:${APP_PORT}"
```

#to run the same as endpoints.sh
`/home/terra/emajprz/backup_and_draft/dev/django-k8s/venv django_k8s.wsgi:application`

## Migration Script:
```
#!/bin/bash

SUPERUSER_EMAIL=${DJANGO_SUPERUSER_EMAIL:-"przemyslaw.majdanski@ericsson.com"}
cd /app/

/opt/venv/bin/python manage.py migrate --noinput
/opt/venv/bin/python manage.py createsuperuser --email $SUPERUSER_EMAIL --noinput || true
```
- we igonore: python manage.py collectstatics as this can be done later (and many times for many app)
- we cannot run migration in docker file as it need to have access to database ;) to work

## Docker Compose part I:

- check used ports: sudo lsof -i -P -n | grep 8000
- docker-compose up & down
- docker-compose up --build
- I faced problem with pull images: docker login && docker pull <image> & docker image 
- generaly creating docker-compose file
- problem with connection between django & posgres solved: When we create docker-compose, name of app: postgres_db must be the same as POSTGRES_HOST from .env. As Django need to have a way to communicate, and docket-compose used this name of app ;) 

## Docker Compose part II:

- `expose` allow connection between pods.
- `port` handle connection with localhost
- to check somethin in shell django:
```
python3 manage.py shell
from django.conf import settings
print(settings.DB_IS_AVAIL)
```

## Differences & additional notes:

CoddingEntrepreneurs use DigitalOceans to privision k8s and posgresql. In our case we will prepare everything in k8s.
Usefull tips:
`export KUBECONFIG=/path/to/kubeconfig.yaml` this will allow to manage k8s
- we can add to workspace confgi file necessary entry so it will automatically load correct kubeconfig file and allow use manage correct k8s, in prod is good to have /.kube and there stored kubeconfig.yaml
- we can find config file in: `less ~/.kube/config`

## Kubernetes:

`kubectl apply -f dev/django-k8s/k8s/deployment.yaml`
`k run nginx-deplo --image=armdocker.rnd.ericsson.se/dockerhub-ericsson-remote/library/nginx:1.23.1-alpine --restart=Never`
`k get deployments nginx -o yaml`
`k get pods`
`k describe deployment <name>`
`k logs <name>`
end on: https://www.youtube.com/watch?v=NAOsLaB6Lfc&t=14254s&ab_channel=CodingEntrepreneurs

## New try on Kubernetes, resetup of this project on new enviroment:

/home/cloud_user/Projects/Djnago-on-k8s/djnago-k8s/dev/django-k8s <- create here venv using:
`python3.8 -m venv venv`
`sudo apt install python3.8-venv` <- solved problem with virtualenv
`source venv/bin/activate `
`pip install pip --upgrade`
`pip install -r requirements.txt`
- I had some problem with installin vnev but finally after reinstall it start to work
`source venv/bin/activate`
`sudo docker-compose up --build`
- there was issue to bring down docker compose and I solved this using: `sudo aa-remove-unknown` (base on: https://stackoverflow.com/questions/47223280/docker-containers-can-not-be-stopped-or-removed-permission-denied-error)
`sudo docker-compose up --build`
`python manage.py shell / makemigrations / migrate / runserver`
`python manage.py shell`
`from django.conf import settings` `print(settings.DB_IS_AVAIL)`
- using docker-compose I can bring up databse posgresql stored on volumes:
      - postgres_data:/var/lib/posgresql/data
- and there is presistent data stored. So in any situation I can bring up django :) seams to be up to date with course. 
- found problem with CoreDNS: I just restarted it: `kubectl rollout restart -n kube-system deployment/coredns` and check logs using: `kubectl logs --namespace=kube-system -l k8s-app=kube-dns`, additional useful website: https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/ and commands to check connectibvity:
    - `kubectl exec -i -t dnsutils -- nslookup kubernetes.default`
    - `kubectl exec pod-svc-test -- curl svc-clusterip`
    
## Start with kubernetes:

`k apply -f k8s/nginx/deployment.yaml` - run base pod
`k exec -it <pod> -- /bin/bas` to open terminal
`k apply -f /home/cloud_user/Projects/Djnago-on-k8s/djnago-k8s/dev/django-k8s/k8s/apps/iac-python.yaml` to run example app on k8s
- on labe nev I found problem with loadbalancing. As it appear to access pod from outside I need to use public ip of worker node, not master one. 
and this seams to be some kind of bug. Interesting article: https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/ from k8s website.
- i used 'NodePort' to expose app to outside world (insted of Loadbalancer from example as he is using digitalocean :) and I am not) 

## Build image and push it out to docker.io:
- to find docker config.json: `sudo less /root/.docker/config.json`
- `sudo docker build -t docker.io/przemaj1990/django-k8s:latest -f Dockerfile . ` build image before push
- `sudo docker push docker.io/przemaj1990/django-k8s --all-tags` push all tags 
- `kubectl create secret docker-registry regcred --docker-username=przemaj1990 --docker-password=<password> --docker-email=przemaj1990@gmail.com` to create secret for k8s to access docker.io
- ` python -c "import secrets; print(secrets.token_urlsafe(32))"` to generate secrets for .env, or you can use django functionality: `python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"` 
- 

## Return to project after moment:
### Again work on network problems:
- `sudo aa-remove-unknown` to resolve peroblem with old not down docker 
- `source venv/bin/activate`
- `python manage.py runserver` 
- I faced some problem with running stadard service + deployment, but iac-python.yaml seams to work correctly:
` k apply -f iac-python.yaml`, and before I also restarted core dns: `kubectl rollout restart -n kube-system deployment/coredns`
- `k get services` or `k get endpoints`
### Build again using private repo:
- to find docker config.json: `sudo less /root/.docker/config.json`
- `sudo docker build -t docker.io/przemaj1990/django-k8s:latest -f Dockerfile . ` build image before push
- `sudo docker push docker.io/przemaj1990/django-k8s --all-tags` push all tags
- `python -c "import secrets;print(secrets.token_urlsafe(32))"` to generate secrets
- `python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"` second option to generate secret key using django utils
- he is setup everything again for new db located externally, so modifing .env file. 
### Kubernetes secrets from env:
- `k create secret generic django-k8s-web-prod-env --from-env-file=.env` to create secrets for k8s
- `k get secrets -o yaml <name>` and when config changed just `k delete secret <name>` and create it again.
### ssl mode:
- `DB_IGNORE_SSL=true` add to .env && necessary config in seetings.
### imagePullSecrets:
- base on: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
- `cat ~/.docker/config.json` && `kubectl create secret generic regcred --from-file=.dockerconfigjson=<path/to/.docker/config.json> --type=kubernetes.io/dockerconfigjson` or we can `kubectl create secret docker-registry regcred --docker-server=<your-registry-server> --docker-username=<your-name> --docker-password=<your-pword> --docker-email=<your-email>`
- `kubectl get secret regcred --output=yaml` and to decode: `kubectl get secret regcred --output="jsonpath={.data.\.dockerconfigjson}" | base64 --decode`
- `k apply -f /home/cloud_user/Projects/Djnago-on-k8s/djnago-k8s/dev/django-k8s/k8s/apps/django-k8s-web.yaml` I started app and ofc faced problem like ALLOWED_HOSTS. 
- `k exec -it <pod name> -- /bin/bash`

## Setup Posgresql on k8s in a way that will allow connection between django&posgrsql:
- First try base on: https://mattermost.com/blog/orchestrate-django-application-with-kubernetes/ 
- I faced problem with: ``` Data page checksums are disabled.

initdb: directory "/var/lib/postgresql/data" exists but is not empty
If you want to create a new database system, either remove or empty
the directory "/var/lib/postgresql/data" or run initdb
with an argument other than "/var/lib/postgresql/data".```
- solved using: https://github.com/docker-library/postgres/issues/263 and it seams that databse start normally
- move to next part so try to migrate data from django to database:
- `k exec -it django-k8s-web-deployment-876654dc5-64qzk -- /bin/bash`
- `source /opt/venv/bin/activate` & `python manage.py migrate` 
- I faced problem with dns, and django not possible to resolve name of postgres: "django.db.utils.OperationalError: could not translate host name "postgres" to address: Temporary failure in name resolution"
- problem was probably on coredns so restart of coredns helped (`kubectl rollout restart -n kube-system deployment/coredns`). Plus I working on thsot using guide: https://kubernetes.io/docs/tasks/administer-cluster/dns-debugging-resolution/. 
- finally I was able to run: `bash migrate.sh` and final migration + be able to log using correct credential into admin account. 
- other usefull site and example: 
    - https://datagraphi.com/blog/post/2021/2/10/kubernetes-guide-deploying-a-machine-learning-app-built-with-django-react-and-postgresql-using-kubernetes
    - https://medium.com/@tech_with_mike/how-to-deploy-a-django-app-over-a-kubernetes-cluster-with-video-bc5c807d80e2
    - https://mattermost.com/blog/orchestrate-django-application-with-kubernetes/
    - https://betterprogramming.pub/how-to-deploy-a-django-application-with-kubernetes-f5814b0688bf

## Deployment Guide: 
- add seperate document: deployment-guide.md in clear way describing steps of deployment.

## GithuAction:
- added action, problem with path: /web that should be the same like on git so dev/django-k8s/web.
- finished at end of : Github Actions  Test Django Automatically
- how to log into dockerhub in github: https://github.com/marketplace/actions/build-and-push-docker-images
- how to add secrets in github: https://github.com/Azure/actions-workflow-samples/blob/master/assets/create-secrets-for-GitHub-workflows.md
- end on 4:15.
- https://github.com/marketplace?type=actions&query=kubectl-action+ <- marketplace with already prepared actions
- I faced probelm with connection to k8s as it is behind proxy/on cloud. So to access it I used https://github.com/marketplace/actions/run-kubectl
with `cat .kube/config` but with changed ip to poing public ip. As effect i faced issue with: 'Invalid x509 certificate for kubernetes master'
wich i solved using: `--insecure-skip-tls-verify` in args sended from git to k8s.
- secrets.KUBE_CONFIG_DATA - setup: `vim kubeconfig-github` && `base64 kubeconfig-github > kubeconfog4`
- force rollout of whole deployment: `kubectl rollout restart deployment/django-k8s-web-deployment`

## Staticfiles using AWS:
- finished at 4:33
