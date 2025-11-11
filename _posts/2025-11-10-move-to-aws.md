---
layout: post
title: "Deployment Live iPXE host moving to AWS"
categories: DeploymentLive ipxe ca cert aws
---

Well, I got a nice little e-mail from Microsoft, letting me know that my Virtual Machine in Azure is being deprecated. Crap.

## Some background:

For the "Free" version of Deployment Live iPXE that was released, we only included ONE custom SSL CA certificate in the signed binary. This was by-design. This means the iPXE client can only connect to a HTTPS server whose key has been signed by that Certificate Authority. 

I tried looking at various options in Azure this summer when I was first setting up the infrastructure for DeploymentLive.com, but I couldn't find any CHEAP Azure Web services that allowed me to bring my own DNS, and my own Cert. 

The best I could do was to create a B1s ( 1 core, 1gb ram ) Windows Virtual Machine running Server core, at about $5/mo. I was silly, the content was just static web pages, but all the other options from Azure were MORE expensive.

Well, Azure sent me an e-mail that they were deprecating my cheap B1s, so time to look for alternatives.

## Enter AWS

I did some Google AI searching, and developed a plan of attack: S3 buckets + CloudFront, + Custom Certs. I think it may be free too 🙂.

So I spent today creating an AWS Account, S3 Bucket, uploading the content, and creating a CloudFront Distribution.

* Still haven't figured out how to auto upload my content to the s3 bucket. Azure had a cool integration with GitHub that auto populated the content after a github push.
* Took me a couple tries to create the S3 bucket with the proper "Open" permissions. I finally had to create a bucket policy that looks like this:
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::deploymentliveweb/*"
        }
    ]
}
```
* Took me several tries to create the right CloudFront Distribution, and I had to create several public/private key pairs signed with my CA to get it to work.
* For whatever reason, CloudFront would NOT allow me to bind the site with my custom Certificate at creation, gave me the error: The certificate that is attached to your distribution was not issued by a trusted Certificate Authority.  Well, no duh, it's not trusted, please use it anyways.  Instead, I ended up having to create the web site with an Amazon generated certificate and then change the bindings later to my imported certificate. 
* Unfortunately, it appears that we can't use the custom port 8050, I was using on the Virtual Machine. 

## Going Forwards

Because we can't use the 8050 port anymore, we have a breaking change, and everyone who was using `boot.deploymentlive.com:8050` should to move to `aws.deploymentlive.com` 

I am in the process updating my own test machines and documentation to reflect this. I hope to have `boot.deploymentlive.com:8050` redirect to another test machine so I don't break things, at least for a short while. 

Tomorrow, I'll test and make the changes to DNS. 

-k