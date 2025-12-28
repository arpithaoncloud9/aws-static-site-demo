
# Static Website Hosting with AWS S3, CloudFront, and Route 53

##  Step 1: Create and Configure Your S3 Bucket

#### Log in to AWS Console 

1. go to **S3** service.
#### Create a new bucket

 1. Name it something unique (e.g., `maria-static-site-demo`).
 2. Choose a region close to you (e.g., `us-east-1`).
 3. Keep **Block Public Access** ON (we’ll use CloudFront for secure access).
 4. **Create your static website files** (HTML, CSS, JS).
    -  Example: `index.html`, `style.css`, `script.js`.
- **index.html 
	``` html
	<!DOCTYPE html>
	<html lang="en">
	<head>
	  <meta charset="UTF-8">
	  <title>Maria's Cloud Project</title>
	  <link rel="stylesheet" href="style.css">
	</head>
	<body>
	  <h1>Welcome to Maria's Cloud Project 🚀</h1>
	  <p>This static website is hosted on AWS S3, delivered securely via CloudFront, and mapped with Route 53.</p>
	  <button onclick="showMessage()">Click Me</button>
	  <p id="output"></p>
	  <script src="script.js"></script>
	</body>
	</html>
	```
- **style.css**
	``` css
	body {
	  font-family: Arial, sans-serif;
	  background-color: #f4f6f8;
	  text-align: center;
	  margin-top: 50px;
	}
	
	h1 {
	  color: #2c3e50;
	}
	
	button {
	  background-color: #3498db;
	  color: white;
	  padding: 10px 20px;
	  border: none;
	  cursor: pointer;
	  margin-top: 20px;
	}
	
	button:hover {
	  background-color: #2980b9;
	}
	```
- **script.js**
	``` js
	function showMessage() {
	  document.getElementById("output").innerText = "Hello from AWS Cloud! 🌐";
	}
	```
#### Test locally

1. Make sure your `index.html` loads correctly in your browser before uploading.
#### Upload the files

1. Upload your static website files to your S3 bucket.
2. Copy your bucket ARN  (e.g.,  arn:aws:s3:::maria-static-site-demo).  //- you’ll need it when creating the CloudFront Origin Access Control (OAC).

## Step 2:  CloudFront Configuration

#### Create an Origin Access Control (OAC)

1. Go to **CloudFront console** → **Origin Access Control** → **Create OAC**.
2. Choose **S3** as the origin type.
3. Give it a name (e.g., `MariaStaticSiteOAC`).
4. Save it.
#### Attach OAC to your CloudFront distribution

1. Click on create CloudFront Distribution.
2. In Origin domain → select your S3 bucket (use the REST API endpoint, not the website endpoint).
3. In Origin settings select:
    - Allow private S3 bucket access to CloudFront – Recommended
    - Customize origin settings
    - Customize cache settings
        - Set to: **Redirect HTTP to HTTPS**
        - Keep it as **GET, HEAD** 
4. This ensures CloudFront is the only service allowed to fetch objects from your bucket.
5. Save and create CloudFront Distribution.
6. Once the CloudFront Distribution is ready:
    - click on your distribution ID → General Settings → Edit → Default root object → index.html (enter file name)
7. Save changes
#### Verify

1. Visit your CloudFront domain (e.g., dldx46bp7p6e1.cloudfront.net) //CloudFront works

<img width="1440" height="646" alt="image" src="https://github.com/user-attachments/assets/21ae9698-9a84-46cd-87a7-1869f78ef7ec" />


2. https://maria-static-site-demo.s3.amazonaws.com/index.html  //S3 URL doesn't work

#### Why CloudFront Works but S3 URL Doesn’t

1. You kept your **S3 bucket private** (Block Public Access ON).
2. That means direct requests to the S3 URL (`https://maria-static-site-demo.s3.amazonaws.com/index.html`) are **blocked**.
3. Only **CloudFront**, with its Origin Access Control (OAC), is allowed to fetch objects from your bucket.
4. So the fact that your CloudFront domain works but the S3 URL doesn’t is proof that your security setup is correct.

## Step 3: Request an SSL Certificate in ACM (us‑east‑1) 

#### Why You Need an SSL Certificate

1. **Secure HTTPS traffic**
    - Without a certificate, your site can only be served over HTTP.
    - With SSL, CloudFront can serve your site over HTTPS, encrypting data between the browser and CloudFront.
    -  This protects users from eavesdropping or tampering.
2. **Custom domain support**
    - CloudFront’s default domain (`dldx46bp7p6e1.cloudfront.net`) already has HTTPS enabled with Amazon’s certificate.
    - If you want to use your own domain (e.g., `setwin.xyz `), you need an ACM certificate for that domain.
#### Open ACM Console

1. In the AWS Management Console, search for **Certificate Manager**.
2. Make sure your region is set to **US East (N. Virginia) – us‑east‑1.** //Even if your S3 bucket is in another region, CloudFront  looks for SSL/TLS certificates in AWS Certificate Manager (ACM) in us‑east‑1 (N. Virginia) . 
3. Click **Request a certificate** → **Request a public certificate**.
4. Enter your domain name(s): (Example: `setwin.xyz`)
5. Add `www.setwin.xyz` as well if you want both root and www versions.
#### Validation Method

1. Choose **DNS validation (recommended)**.
2. This is easier because you’ll add a CNAME record in Route 53 (or your DNS provider).
3. CloudFront will automatically detect validation once the record is added.
4. Click request
## STEP 4: Use Route 53 With a Domain Registered 

#### Create a Hosted Zone in Route 53

1. Go to **AWS Console → Route 53 → Hosted Zones → Create Hosted Zone**.
2. Enter your domain: `setwin.xyz`.
3. Choose **Public Hosted Zone**.
4. Route 53 will generate 4 **Name Server (NS) records** automatically.
#### **Update Name Servers in Registrar**

 1. If your domain is registered with a third‑party registrar (e.g., GoDaddy, Namecheap, Spaceship):
 2. Log in to Registrar → Domain Management → DNS/Nameservers.
 3. Replace Registrar’s default name servers with the 4 Route 53 name servers.
 4. Save changes.
#### Create Your DNS Records in Route 53

1.  **ACM Validation CNAME** 
-   Go to Route 53 → Hosted Zone → `setwin.xyz` → **Create record** and add:
    - Name: **CNAME name** (from ACM) // make sure no domain duplicates
    - Type: `CNAME`
    - Value: **CNAME value**

1.  **A Record for Root Domain**
    - Name: _(blank)_ (represents `setwin.xyz`)
    - Type: `A – Alias` 
    - Alias target: your CloudFront distribution (`dldx46bp**871.cloudfront.net`)

2.  **For Subdomain** (Optional)
    - Name: `www`
    - Type: `A – Alias`
    - Value: dldx46bp7p6e1.cloudfront.net (`dldx46bp**871.cloudfront.net`)
    - Save the records and allow 5–30 minutes for DNS propagation.
    - Once validated, ACM will mark the certificate as **Issued**, and you can attach it to your CloudFront distribution.

3. **DNS Propagation Check** 
    - Enter: `_5899e95f7243bc637d8b55tgffdghvb.setwin.xyz`
    - Use https://dnschecker.org:
    - Type: `CNAME`
    - Confirm that the value appears globally
#### **Attach Certificate to CloudFront**

1. Once ACM status shows **Issued**, go to **CloudFront → Settings → General.**
2. Add **Alternate Domain Names (CNAMEs)**:
    - `setwin.xyz`
    - `www.setwin.xyz`

3. Select your ACM certificate.
4. Save
## Step 5: Verify Your Site

 1. Wait for DNS propagation.
 2. Visit `https://setwin.xyz` // your static site should load securely via CloudFront.
 3. Visit `https://www.setwin.xyz` // should redirect to the same site if you added the CNAME.
 4. Confirm the 🔒 padlock in your browser.

<img width="1110" height="341" alt="setwin xyz_domain" src="https://github.com/user-attachments/assets/7f3e9f10-da30-4904-83d4-2071f583c90a" />


<img width="1117" height="315" alt="www setwin xyz" src="https://github.com/user-attachments/assets/2e6534e4-fdf4-489d-a8c5-9bc353734f4b" />

