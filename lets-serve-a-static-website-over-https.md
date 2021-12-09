# Let's Serve a Static Website Over HTTPS

As an IAM User sign in to AWS Management Console.

### Create a hosted zone for your domain in Route 53

1. Go to [Route 53 Dashboard](https://console.aws.amazon.com/route53).
2. Select **Hosted zones**.
3. Click **Create hosted zone**.
4. Fill in **Domain name** input with your domain name.
5. Click **Create hosted zone**.
6. Update Nameservers values of your domain on the DNS registrar of your choice (record of type **NS**, make sure addresses contain periods at ends).
7. Take a :coffee:break and use [DNS checker](https://dnschecker.org/#NS) to check how is DNS propagation going.

### Create an SSL certificate for your domain

1. Go to A[WS Certificate Manager Dashboard](https://console.aws.amazon.com/acm/home?region=us-east-1), make sure the **N.Virginia** region is selected (CloudFront requires certificates created in this region).
2. Click **Request**.
3. Select **Request a public certificate** and click **Next**.
4. Fill in **Fully qualified domain name** input with your domain name.
5. Leave **DNS validation - recommended** selected and click **Request**.

A record of type **CNAME** will be added to your hosted zone. It's a type of record that maps one domain name to another.

### Create S3 buckets

1. Go to [Amazon S3 Dashboard](https://s3.console.aws.amazon.com/s3/home).
2. Click **Create bucket**.
3. Fill in **Bucket name** input with a unique name, this bucket will store static files of your website.
4. Select desired **AWS Region**.
5. Leave the rest defaults selected and click **Create bucket**.
6. Upload a simple index.html file to the bucket, an example you can find [here](https://raw.githubusercontent.com/annalach/cdn-on-aws-static-website/main/.gitbook/assets/index.html).
7. Proceed to create another bucket, this one will be used to store logs.
8. Fill in **Bucket name** and select **AWS Region**.
9. This time, select **ACLs enabled**.
10. Leave the rest defaults selected and click **Create bucket**.

**S3 ACL (access control list)** is a legacy access control mechanism. It defines which AWS accounts or groups are granted access and the type of access. As a general rule, AWS recommends using S3 bucket policies or IAM policies for access control. &#x20;

### Create your CloudFront distribution

1. Go to [Amazon CloudFront Dashboard](https://console.aws.amazon.com/cloudfront/v3/home).
2. Click **Create distribution**.
3. As **Origin domain**, select your S3 bucket's domain.
4. Select **Yes use OAI**.
5. Click **Create new OAI** and **Create**.
6. Select **Yes, update the bucket policy**.
7. Select **Redirect HTTP to HTTPS**.
8. Under **Alternate domain name (CNAME) - optional**, click **Add item** and fill in input with your domain name.
9. In **Custom SSL certificate - optional**, select your SSL certificate created with AWS Certificate Manager.
10. Type **index.html** in **Default root object - optional** input.
11. Select **On** under **Standard logging** and select your S3 bucket.
12. Leave the rest defaults selected and click **Create distribution**.

CloudFront distribution needs a few minutes to be deployed.

**OAI** stands for **Origin Access Identity** is a special CloudFront user, when using OAI only CloudFront can access files in your bucket. Go to your origin bucket and see the bucket's policy.

Check the ACL of your logs bucket. Read and Write access has been granted to the AWS account with ID `c4c1ede66af53448b93c283ce9448c4ba468c9432aa01d700d3878632f77d2d0`\
it's the `awslogsdelivery` account,

### Route traffic from your domain to your CloudFront distribution

1. Go to [Route 53 Dashboard](https://console.aws.amazon.com/route53/v2/home) and details of your hosted zone.&#x20;
2. Click **Create record**.
3. Select **Record type** **A** and switch on **Alias**.
4. In **Route traffic to** select **Alias to CloudFront distribution**.
5. Select your distribution.
6. Click **Create records**.

The **A** record type is the most fundamental type of DNS record. It indicates the IP address of a given domain. **Amazon Route 53 alias record** is a Route 53-specific extension to DNS functionality. It lets you route traffic to selected AWS resources, such as CloudFront distribution in our case.

Wait a couple of minutes and try to visit your domain in a web browser. You should see your index.html page.

### Add 404 page

If you try to visit a page that doesn't exist on your site you will see an ugly 403 XML error. Let's add a simple 404 page. You can find an example [here](https://raw.githubusercontent.com/annalach/cdn-on-aws-static-website/main/.gitbook/assets/404.html).&#x20;

1. Upload a 404.html file to your origin S3 bucket.
2. Go to [Amazon CloudFront Dashboard](https://console.aws.amazon.com/cloudfront/v3/home) and details of your distribution.
3. Select **Error pages** tab.
4. Click **Create custom error response**.
5. In **HTTP error code**, select **403: Forbidden**.
6. Select **Yes** under **Customize error response**.
7. Type **/404.html** in **Response page path**.
8. Select **404: Not Found** under **HTTP Response code**.
9. Click **Create custom error response**.

404 page should be returned instead of the ugly 403 XML once changes are deployed.
