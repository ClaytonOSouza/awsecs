#!/usr/bin/env bash
DOCKER_REPO=`aws ecr describe-repositories — repository-names  sitemeu| grep repositoryUri | cut -d “\”” -f 4`
docker build — no-cache -t ${DOCKER_REPO}:1.0 .
docker push ${DOCKER_REPO}:1.0
