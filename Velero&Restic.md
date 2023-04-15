# Velero&Restic-k8s

Typical commands that might be useful when installing Velero with Restic on a K8s cluster (local, in my case)

***Download and copy  Velero binary file (Ubuntu):***

	test ! -d /tmp/velero/ && mkdir /tmp/velero/ || rm -rf /tmp/velero/ && mkdir /tmp/velero/; wget -qO- https://github.com/vmware-tanzu/velero/releases/download/v1.9.6/velero-v1.9.6-linux-amd64.tar.gz | tar zxvf - -C /tmp/velero/; sudo mv /tmp/velero/*/velero /usr/local/bin

***Create S3 bucket for backups ("k8s-backups-home" in my case, located in the "us-east-1" region):***

    aws s3api create-bucket \
    --bucket k8s-backups-home \
    --region us-east-1

***Create IAM user (velero in my case):***

    aws iam create-user --user-name velero

***Create S3 Policy for the user:***

    cat > velero-policy.json <<EOF
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "ec2:DescribeVolumes",
                    "ec2:DescribeSnapshots",
                    "ec2:CreateTags",
                    "ec2:CreateVolume",
                    "ec2:CreateSnapshot",
                    "ec2:DeleteSnapshot"
                ],
                "Resource": "*"
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:GetObject",
                    "s3:DeleteObject",
                    "s3:PutObject",
                    "s3:AbortMultipartUpload",
                    "s3:ListMultipartUploadParts"
                ],
                "Resource": [
                    "arn:aws:s3:::k8s-backups-home/*"
                ]
            },
            {
                "Effect": "Allow",
                "Action": [
                    "s3:ListBucket"
                ],
                "Resource": [
                    "arn:aws:s3:::k8s-backups-home"
                ]
            }
        ]
    }
    EOF

***Attach the policy to the user:***

    aws iam put-user-policy \
      --user-name velero \
      --policy-name velero \
      --policy-document file://velero-policy.json
***Create an access key for the user:***

    aws iam create-access-key --user-name velero

*The result should look like:*

    {
      "AccessKey": {
            "UserName": "velero",
            "Status": "Active",
            "CreateDate": "2023-03-26T18:31:28.348Z",
            "SecretAccessKey": *******<AWS_SECRET_ACCESS_KEY>*******,
            "AccessKeyId": *******<AWS_ACCESS_KEY_ID>*******
      }
    }

***Create S3 credentials file (.s3-creds or .minio-creds) in a local directory with such format:***

    [default]
    aws_access_key_id=*******<AWS_SECRET_ACCESS_KEY>*******
    aws_secret_access_key=*******<AWS_ACCESS_KEY_ID>*******

***Install Velero with Restic to the K8s cluster to work with AWS S3 bucket:***

    velero install \
        --image velero/velero:v1.9.6 \
        --provider aws \
        --plugins velero/velero-plugin-for-aws:v1.6.0 \
        --use-volume-snapshots=false \
        --bucket k8s-backups-home \
        --backup-location-config region=us-east-1 \
        --secret-file ~/.s3-creds \
        --use-restic

***or with Minio S3 storage:***

    velero install \
        --image velero/velero:v1.9.6 \
        --provider aws \
        --plugins velero/velero-plugin-for-aws:v1.6.0 \
        --bucket k8s-backups-home \
        --use-volume-snapshots=false \
        --secret-file ~/.minio-creds \
        --use-restic \
        --backup-location-config region=default,s3ForcePathStyle="true",s3Url="http://***************.site"

***Create backup ("jenkins-home") of the namespace ("jenkins" in my case) with persistent data:***

    velero backup create jenkins-home --include-namespaces jenkins --default-volumes-to-restic --wait

***Show content of the backup "jenkins-home":***

    velero backup describe jenkins-home --details

***Restore of the backup "jenkins-home" to the original namespace:***

    velero restore create --from-backup jenkins-home --wait

***Create backup of the whole clluster (not recomended due to known issues with restore using selectors https://github.com/vmware-tanzu/velero/issues/1970 ):***

    velero backup create k8s-backups-home --default-volumes-to-restic --wait

***Sync S3 storage to tge local folder using AWS CLI with removing irrelevant files in local folder if required (will require more wide Policy) :***

    aws s3 sync s3://k8s-backups-home . --delete
