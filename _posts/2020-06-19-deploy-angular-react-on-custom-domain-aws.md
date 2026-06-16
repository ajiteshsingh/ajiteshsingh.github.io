---
layout: post
title: "Deploy Angular/React website on a custom domain using AWS S3, Cloudfront and route 53"
date: 2020-06-19 12:00:00 -0700
categories: [devops, aws]
tags: [aws, s3, cloudfront, route53, angular, react, deployment, dns]
---

If you are here, I am pretty sure you already have an Angular/React application ready to be deployed and a domain registered with any of the domain registrars. In this article we will be deploying a domain registered on godaddy. I am sure the process would be the same for other domain registrars as well. 

Some prerequisites..

**What is DNS(Domain Name Server) ?**

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image1.png" | relative_url }})

We know that each device connected to the internet has a public IP address. DNS is the phonebook of the internet. It keeps mapping of domain names to its ip address which domain names like google.com, facebook.com is easier for humans to remember than IP address which are just a sequence of numbers. **Route53** is a highly scalable DNS web service provided by AWS.

**What is S3 ?**  
S3 (Simple storage solution) which is also an AWS service is used to store any file/object on the cloud through a web service interface or programmatically. There is the concept of buckets in s3 which is just like creating folders. One thing to keep in mind is whenever you create a bucket it must have a unique name across all the s3 buckets available on the internet and you need to select a region where you want to store the bucket. More on this later.

**What is Cloudfront ?**

One drawback of s3 is if the bucket is hosted in the US and someone from Asia is trying to access the content of that bucket. It will take a lot of time for the content to load since the content has to take a lot of hopes before reaching asia. This problem is solved using CDN(Cloudfront Distribution Network). Cloudfront Distribution Network caches the content of your s3 bucket in different regions. So whenever you try to access any content of s3 via CDN, CDN first checks if the content is cached in the edge region, serves it if already available. And if the content is not cached yet, it will fetch the content from the original source and cache it for further use.

**How Exactly is my website served via cloudfront, route 53 and s3 ?**

Before we get into the steps it’s important to understand the flow of how your website is actually served. If you understand this flow, Trust me it wouldn’t take more than 30mins to complete the entire steps. I personally struggled a lot while doing this because there are some critical things which you need to take care of and none of the blogs I read mentioned it.

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image2.png" | relative_url }})

To start in simple terms just look at the image above. S3 stores the code of your website. Cloudfront is a way to cache and load your files stored from s3 faster. Route53 is the DNS service which maps your domain name to server ip address which then fetches the content from the cloudfront and sends back the response.

So when you type [www.](http://www.mywebsite.com)[mywebsite](http://www.mywebsite.com)[.com](http://www.mywebsite.com) in the url. This is what happens

1. Request is sent to the DNS server which resolves the hostname ([www.mywebsite.com](http://www.mywebsite.com)) into a Public IP address. ([121.51.155.50](http://121.51.155.50/))  
2. DNS routes the request to Cloudfront POP (Edge Location).  
3. In the Edge Location, Cloudfront checks it’s cache if the requested file is available. If the file exists, cloudfront returns it to the requested client machine. If the file is not present in the cache, then cloudfront forwards the requests to Amazon S3, fetches the content from s3 and returns to the user. Also caches the content for future requests.

Now let’s deploy our website. I will be using [www.yourwebsite.com](http://www.arrayofcode.com) as an example custom domain where we want to deploy our frontend application.

**Step 1:**

We will create two s3 buckets with the same name as your domain name. Make sure to select US east (N virginia) as the region for the buckets and deselect Block public access option. The name of the two buckets would look like this

1. [www.yourwebsite.com](http://www.arrayofcode.com)  
2. yourwebsite.com

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image3.png" | relative_url }})

Click on the Create bucket. It will open up a popup something like shown below. Enter your bucket name For eg. www.yourwebsite.com . Select the region as US East (N. Virginia). Don’t touch the copy settings. Click Next

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image4.png" | relative_url }})

Next you will see the configure options screen. Keep ths default configuration, don’t change anything. Click next.

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image5.png" | relative_url }})

Next is setting the permissions for the bucket. This is important. By default the block all public access button will be checked, you need to **uncheck** it and click on the i acknowledge button.

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image6.png" | relative_url }})  
Finally in step 4 just review the options once and click on create bucket

Repeat this process twice. Once with the bucket name [www.yourwebsite.com](http://www.yourwebsite.com) and once with bucketname as yourwebsite.com

**Step 2: Giving the bucket public access and enabling static website hosting**

Once both the buckets are created. We need to give public read access to both the buckets.

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image7.png" | relative_url }})

Go inside the bucket. Select Permissions and then select bucket policy.  
Copy the policy below making sure you change the YOUR\_BUCKETNAME with your bucket name Eg. [www.yourwebsite.com](http://www.yourwebsite.com) or yourwebsite.com. Paste it in the textbox and click save

{  
    "Version": "2008-10-17",  
    "Statement": \[  
        {  
            "Sid": "AllowPublicRead",  
            "Effect": "Allow",  
            "Principal": {  
                "AWS": "\*"  
            },  
            "Action": "s3:GetObject",  
            "Resource": "arn:aws:s3:::YOUR\_BUCKETNAME/\*"  
        }  
    \]  
}

Follow the above steps for both the buckets. The objects in the bucket are now publicly accessible. 

Let’s now enable static website hosting on both the buckets

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image8.png" | relative_url }})

This step is going to be different for both the buckets. 

* For bucket with www as the prefix. 

Go to properties Select Static Website hosting. Click on the use this bucket to host a website. Fill in index.html in index document and index.html in error document. (If you have a custom error page you can enter that filename.html). For now I would use index.html for both index document and error document. Click save

* For the bucket without the prefix of www.

Go to properties. Select Static Website hosting. Click on “use this bucket to host a website”. Select redirect requests and enter [www.yourwebsite.com](http://www.yourwebsite.com) in the bucket name. Click save.

So far we have created two buckets, given public read object access and enabled static website hosting on it. Make sure the bucket shows a yellow public access besides the bucket name.

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image9.png" | relative_url }})

**Step 3 \- uploading website on s3**

If you are deploying an Angular application. Run ng build \--prod. This will create a dist folder which has the productionised code. Open [www.yourwebsite.com](http://www.yourwebsite.com) bucket and upload all the files in the root bucket itself. 

Note:- upload all the files and not the folder. The root directory must have index.html and all the files.

The bucket should look something like this.  
![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image10.png" | relative_url }})

Awesome\! Your website is deployed on s3 now. Go inside your [www.yourwebsite](http://www.yourwebsite).com bucket. In the properties select static website hosting. You will find an endpoint, copy the link and open in the new tab.  
![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image8.png" | relative_url }})

If your website loads, you have done everything correctly so far. And if it doesn’t you must have missed something, just read the above steps again and check if you missed anything.

**Step 4 \- Request a certificate if you have bought your domain from godaddy or other domain registrars.** 

If you have registered your domain other than via route 53 than it is important to verify that the domain belongs to you.

Go to Certificate manager in AWS Services. Click on request for certificate and select public certificates. In the add domain name add two domains   
yourwebsite.com  
\*.yourwebsite.com  
The \* signifies any sub domains within your domain

Click Next  
Select the DNS Validation.  
Add tags for your own reference, it’s optional  
Review it once and the click confirm and request  
You will see a Export DNS configuration to a file. Download the file  
It should have the following entries  
![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image11.png" | relative_url }})  
Now login to the domain registrar account. In my case it is godaddy. Select the domain name and go to Manage DNS Option.  
It should look something like this   
![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image12.png" | relative_url }})

Click on ADD, select a CNAME Record from the dropdown. You will see two text inputs with the labels Host and Points to. In the csv exported file from route53, copy the first Record Name but exclude the domain name from it. For eg the record name will show something like this \_34c5df753f510e5911f322d1066aeb24.yourwebsite.com. Just copy \_34c5df753f510e5911f322d1066aeb24 this and paste it in the host\*. Next copy the record value without the last period into the Points to input field. For eg. in my csv file the record value is \_e47dd09b646f951f2a687219bbb63077.tfmgdnztqk.acm-validations.aws.  
Just paste \_e47dd09b646f951f2a687219bbb63077.tfmgdnztqk.acm-validations.aws (Notice the last dot is removed)

Do this twice for each row in the downloaded csv. This means adding two CNAME records. Once you are done with this. Go back to AWS certificate manager, within a couple of minutes the status in the certificate manager should change from pending to Issued. 

Good job\! Now AWS knows that you are the owner of this domain. 

**Step 5 \- Creating cloudfront distribution for s3 buckets**

Go to AWS services and search for cloudfront.  
Click on create distribution  
Select Get started for Web  
The origin domain name will give options to select the bucket. DON’T SELECT FROM THE DROPDOWN. I myself struggled for more than half a day to figure out this. So the problem is if you select from the dropdown your home page of the website will load perfectly fine. But if you try to go to some subroutes directly from url it gives access denied error. For eg. [www.yourwebsite.com](http://www.yourwebsite.com) will load but if you go directly to [www.yourwebsite.com/different\_page](http://www.yourwebsite.com/different_page) it gives access denied.   
So you need to manually enter the origin domain name

Enter this in the origin domain name   
YOUR\_BUCKETNAME.s3-website-BUCKET\_REGION.amazonaws.com

Replace YOURBUCKETNAME and BUCKET\_REGION with your bucketname and region of the bucket

In my case the url looks something like this (Keep in mind we are creating cloudfront distribution for the bucket with www in the prefix)   [www.yourwebsite.com.s3-website-us-east-1.amazonaws.com](http://www.yourwebsitecom.s3-website-us-east-1.amazonaws.com)

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image13.png" | relative_url }})

As soon as you enter the origin domain name most of the other fields will get prefilled.  
Next in viewer protocol policy select redirect http to https

Enter your domain domain in the Alternate Domain names (Again just to double check we are doing this process for the bucket with www as prefix, hence add [www.yourwebsite.com](http://www.yourwebsite.com))

Select Custom SSL, It will show a dropdown with all the domains you have verified using the certificate manager. Select the domain you want to deploy on.  
![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image14.png" | relative_url }})

Next in Default root object enter index.html

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image15.png" | relative_url }})

That's all, now click on create distribution and grab a cup of coffee. It takes around 10-15 minutes for cloudfront to create distribution. 

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image16.png" | relative_url }})

After the cloudfront is deployed it will show an Enabled state besides the cloudfront name.

Copy the text below domain name (xxxxxx.cloudfront.net), open the link in a new tab. If your web application loads, congratulations, the major part of the process is done. Now we just need to load this cloudfront url for our custom domain

Repeat the same process for the bucket without the ‘www’  prefix  Finally you should have two cloudfront distributions. 

**Step 6 \- FInal steps.. Adding hosted zone in route 53 and adding the server names in godaddy account**

In Your AWS services go to route 53, Select hosted zones from the left panel. Click on create hosted zones. On the right hand side you will see an option to enter domain name. Add your domain name (DO NOT ADD www prefix here only the domain name) For eg. yourwebsite.com

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image17.png" | relative_url }})

Click create

Two record sets will be created automatically. One will be of type NS (Name Server) with four values and the other one will be of type SOA (Start of Authority) record.   
Now we need to add the Name servers in our godaddy DNS configurtion. Login again to your respective domain registrars account. In my case it’s godaddy and go to manage DNS option. 

You will see a NameServers option which says default nameservers. Click on Change.

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image18.png" | relative_url }})

Click on Enter my own Nameservers

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image19.png" | relative_url }})

Add all the four Name server values from AWS hosted zones for your domain  
![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image20.png" | relative_url }})

Click on save.

It will take about a minute for the changes to reflect. Once it’s done. You will see using custom nameserver below the NameServer.

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image21.png" | relative_url }})

Go back to route53 in the hosted zones,

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image22.png" | relative_url }})

Select the domain name. Click on create record set. Enter www besides the domain name. Select yes in the Alias. Click on the input box it will give a dropdown, select the cloudfront distribution for [www.yourwebsite.com](http://www.yourwebsite.com) bucket and click create.

Create one more record set. Keep the Name field empty this time and select yes in alias. From the dropdown select the cloudfront url for ‘yourwebsite.com’ (bucket without the ‘www’ prefix) bucket and click create.

![]({{ "/assets/deploy-angular-react-on-custom-domain-aws/image23.png" | relative_url }})

That's all, you are done. Open up a new tab and go to your website (Sometimes it might take some time for the website to load on the custom domain but shouldn't take more than half an hour).

If the article was helpful do follow us on linked in for more such article updates.
