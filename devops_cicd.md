---
layout: default
title: "Devops CoreConcepts"
permalink: /ansible-desired-state.html
---

# DevOps Continuous Integration and Delivery: Setting up a CI/CD Pipeline

## Verify the Lab Environment
```
watch "ps -ef | grep gitlab-ce"
watch "sudo gitlab-ctl status"
sudo docker version
terraform version
sudo gitlab-runner status
hostname
cat /etc/hosts
# The important lines are the four at the end, which map host names to IPs that will be used during the lab:
# 127.0.0.1 gitlab.example.com
# 127.0.0.1 registry.example.com
# 172.31.24.30 gitlab.example.com
# 172.31.24.30 registry.example.com
sudo gitlab-ctl reconfigure
sudo cat /etc/gitlab/initial_root_password #to get password for root user of gitlab
# login on http://gitlab.example.com
ssh-keygen
cd /home/pslearner/.ssh/
cat id_rsa.pub #and copy to gitlab & create public repo
cd ~
git config --global user.name "PS Learner"
git config --global user.email "pslearner.example.com"
```
Lastly, you will set up your gitlab runner  click Settings > CI/CD
Runners > click Create project runner.
In the Terminal, type in sudo (note the space) then paste in the command you just copied. You can right-click in the Terminal and choose Paste.
When prompted for the URL, accept the default by pressing enter.
For the runner, enter runner1.
For the executor, enter docker.
Finally, for the Docker image, enter alpine:latest. This image was downloaded for you when the lab was being set up.
`sudo vim /etc/gitlab-runner/config.toml`:
Under the runners section at the very end of the file, add the following two lines:

```
extra_hosts = ["gitlab.example.com:172.31.24.30"]    
    pull_policy = "never"
```
and then: `sudo systemctl restart gitlab-runner` and `sudo systemctl status gitlab-runner`

## Implement a Unit Testing Stage
```
cd /home/pslearner/lab/
cp -rv * /home/pslearner/ci-cd/
ls -a /home/pslearner/ci-cd/
cd /home/pslearner/ci-cd/ci_files
cp gitlab-ci-test.yml ../.gitlab-ci.yml
cd ..
cat .gitlab-ci.yml
```
gitlab-ci.yml:
```
image: ubuntu/apache2

stages:
  - build
  - test

variables:
  WEBROOT: /var/www/html
  APACHE_CONF_DIR: /etc/apache2/sites-available/

build:
  stage: build
  script:
    - mkdir -p ${WEBROOT}
    - cp index.html ${WEBROOT}/index.html
    - cp style.css ${WEBROOT}/style.css
    - echo "DocumentRoot ${WEBROOT}" >> ${APACHE_CONF_DIR}/000-default.conf

test:
  stage: test
  script:
    - curl -s ${WEBROOT}/index.html > /dev/null || echo "Test failed"
    - cat ${WEBROOT}/index.html | grep "<h1>" > /dev/null || echo "Test failed"
    - curl -s ${WEBROOT}/style.css > /dev/null || echo "Test failed"
```
and then:
```
git add -A
git commit -m "First commit"
git push
```
## Implement a Deploy Stage

`cp ci_files/gitlab-ci-deploy.yml .gitlab-ci.yml`
cat .gitlab-ci.yml:
```
image: ubuntu/apache2

stages:
  - build
  - test
  - deploy

variables:
  WEBROOT: /var/www/html
  APACHE_CONF_DIR: /etc/apache2/sites-available/

build:
  stage: build
  script:
    - mkdir -p ${WEBROOT}
    - cp index.html ${WEBROOT}/index.html
    - cp style.css ${WEBROOT}/style.css
    - echo "DocumentRoot ${WEBROOT}" >> ${APACHE_CONF_DIR}/000-default.conf

test:
  stage: test
  script:
    - curl -s ${WEBROOT}/index.html > /dev/null || echo "Test failed"
    - cat ${WEBROOT}/index.html | grep "<h1>" > /dev/null || echo "Test failed"
    - curl -s ${WEBROOT}/style.css > /dev/null || echo "Test failed"

deploy:
  stage: deploy
  script:
    - cp index.html ${APACHE_CONF_DIR}/index.conf
    - service apache2 restart
```

