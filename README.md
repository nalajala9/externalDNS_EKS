# Creation of External DNS which will help us to automatically create , update and delete the records in Route53.

> Note: Make sure that a domain name and a Route 53 hosted zone exist.

## Step-1: Set up IAM permissions to give the ExternalDNS pod permissions to create, update, and delete Route 53 records in your AWS account.

* Name of the Policy : AllowExternalDNSUpdates(u can provide your custom name)

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "route53:ChangeResourceRecordSets"
      ],
      "Resource": [
        "arn:aws:route53:::hostedzone/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "route53:ListHostedZones",
        "route53:ListResourceRecordSets"
      ],
      "Resource": [
        "*"
      ]
    }
  ]
}

```
> Make a note of Policy ARN which we will use in next step

## Step-2: Create an IAM role for the service account using the preceding policy:

```
# Template
eksctl create iamserviceaccount \
    --name service_account_name \
    --namespace service_account_namespace \
    --cluster cluster_name \
    --attach-policy-arn IAM_policy_ARN \
    --approve \
    --override-existing-serviceaccounts
    
```
*  Note: Replace SERVICE_ACCOUNT_NAME with your service account's name, NAMESPACE with your namespace, CLUSTER_NAME with your cluster's name, and IAM_POLICY_ARN with your IAM policy's ARN.

> Verify the Service Account

     kubectl get sa

## Step-3: Deploy ExternalDNS

* Check if RBAC is enabled in your Amazon EKS cluster:

      kubectl api-versions | grep rbac.authorization.k8s.io

* Note: Before you apply the following manifests, check for the latest version of ExternalDNS (from the GitHub website).

      https://github.com/kubernetes-sigs/external-dns/releases
  
* If RBAC is enabled, then use the above manifest DeployExternalDNS.yaml to deploy ExternalDNS:
   
       kubectl apply -f DeployExternalDNS.yaml
   
*  Verify that the deployment was successful:

       kubectl get deployments

* You can also check the logs to verify that the records are up to date:

      kubectl logs -f $(kubectl get po | egrep -o 'external-dns[A-Za-z0-9-]+')









 
