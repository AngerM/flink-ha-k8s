*** NOTE THIS REPO IS NO LONGER MAINTAINED, BUT IT SHOULD PROVIDE YOU WITH A DECENT START ***

# flink-ha-k8s
An example of how to build modern flink in ha mode on AWS w/ k8s

You will need an external zookeeper quorum to run in HA mode. You can either use zetcd or create an actual zookeeper quorum (either also in k8s or with something like EMR).

These files are forked from the open source flink repo, because those didn't work for me and most of the documentation I found seemed out of date (EX: the images the default flink Dockerfile builds won't work with S3 or kinesis). The docker file also adds the necessary files to allow S3 access and checkpointing by default and switches to debian stretch to resolve library issues with the flink kinesis add-on.

This has been tested and deployed with 1.8.1 w/ Hadoop 2.8 reading+writing to kinesis streams

# How to use it
Build the docker image first w/ the build.sh script in docker folder.
Something like this:
```
./build.sh --from-release --flink-version 1.8.1 --scala-version 2.12 --hadoop-version 2.8 --image-name flink-181-my-job --job-artifacts my_job.jar
```

Then push this to your ECR repo

Afterwards go to the k8s repo and use something like envsubts to template the files. You will need to push up (separately) a k8s secret with the AWS creds to access whatever streams/s3 buckets you want your job to use.

```
FLINK_IMAGE_NAME=<ECR_IMAGE_NAME> FLINK_JOB_PARALLELISM=3 S3_BUCKET=<MY_BUCKET> ZK_QUORUM=<IP:PORT comma separated list> AWS_SECRETS=<MY_K8S_SECRET_WITH_AWS_CREDS> envsubst < task-manager-deployment.yaml.template > task-manager-deployment.yaml
FLINK_IMAGE_NAME=<ECR_IMAGE_NAME> FLINK_JOB_PARALLELISM=3 S3_BUCKET=<MY_BUCKET> ZK_QUORUM=<IP:PORT comma separated list> AWS_SECRETS=<MY_K8S_SECRET_WITH_AWS_CREDS> envsubst < job-cluster-job.yaml.template > job-cluster-job.yaml
```

Then apply the 3 files:
```
kubectl apply -f job-cluster-service.yaml
kubectl apply -f job-cluster-job.yaml
kubectl apply -f task-manager-deployment.yaml
```
