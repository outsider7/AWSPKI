

# Introduction

In this workshop, we will teach you how to securely setup a complete CA hierarchy using AWS Private Certificate authority (AWS Private CA) and create certificates for various use cases. These use cases include internal applications that terminate TLS, code signing, document signing, IoT device authentication, and email authenticity verification. We will cover job functions such as CA administrators, application developers and security administrators and show you how these personas can follow the principle of least privilege and perform various functions associated with certificate management. In addition, you will also learn how to monitor your PKI infrastructure using AWS Security Hub.

[

## Duration![Header anchor link](data:image/svg+xml,%3C%3Fxml%20version%3D%221.0%22%20encoding%3D%22UTF-8%22%3F%3E%3C!DOCTYPE%20svg%20PUBLIC%20%22-%2F%2FW3C%2F%2FDTD%20SVG%201.1%2F%2FEN%22%20%22http%3A%2F%2Fwww.w3.org%2FGraphics%2FSVG%2F1.1%2FDTD%2Fsvg11.dtd%22%3E%3Csvg%20fill%3D%22%23000000%22%20xmlns%3D%22http%3A%2F%2Fwww.w3.org%2F2000%2Fsvg%22%20xmlns%3Axlink%3D%22http%3A%2F%2Fwww.w3.org%2F1999%2Fxlink%22%20version%3D%221.1%22%20width%3D%2224%22%20height%3D%2224%22%20viewBox%3D%220%200%2024%2024%22%3E%3Cpath%20d%3D%22M10.59%2C13.41C11%2C13.8%2011%2C14.44%2010.59%2C14.83C10.2%2C15.22%209.56%2C15.22%209.17%2C14.83C7.22%2C12.88%207.22%2C9.71%209.17%2C7.76V7.76L12.71%2C4.22C14.66%2C2.27%2017.83%2C2.27%2019.78%2C4.22C21.73%2C6.17%2021.73%2C9.34%2019.78%2C11.29L18.29%2C12.78C18.3%2C11.96%2018.17%2C11.14%2017.89%2C10.36L18.36%2C9.88C19.54%2C8.71%2019.54%2C6.81%2018.36%2C5.64C17.19%2C4.46%2015.29%2C4.46%2014.12%2C5.64L10.59%2C9.17C9.41%2C10.34%209.41%2C12.24%2010.59%2C13.41M13.41%2C9.17C13.8%2C8.78%2014.44%2C8.78%2014.83%2C9.17C16.78%2C11.12%2016.78%2C14.29%2014.83%2C16.24V16.24L11.29%2C19.78C9.34%2C21.73%206.17%2C21.73%204.22%2C19.78C2.27%2C17.83%202.27%2C14.66%204.22%2C12.71L5.71%2C11.22C5.7%2C12.04%205.83%2C12.86%206.11%2C13.65L5.64%2C14.12C4.46%2C15.29%204.46%2C17.19%205.64%2C18.36C6.81%2C19.54%208.71%2C19.54%209.88%2C18.36L13.41%2C14.83C14.59%2C13.66%2014.59%2C11.76%2013.41%2C10.59C13%2C10.2%2013%2C9.56%2013.41%2C9.17Z%22%20%2F%3E%3C%2Fsvg%3E)

](https://catalog.workshops.aws/certificatemanager/en-US/introduction#duration)

The core sections of the workshop (CA Hierarchy Setup and Internal Application Exercise) should take approximately one hour. You can then choose to complete additional exercises ranging from 30 minutes to one hour long each.

[

## Audience

](https://catalog.workshops.aws/certificatemanager/en-US/introduction#audience)

The target audience of this workshop is anyone who is interested to learn about PKI on AWS. This includes, but is not limited to: security professionals, PKI operators, application developers, and architects.

[

## Background Knowledge

](https://catalog.workshops.aws/certificatemanager/en-US/introduction#background-knowledge)

This workshop may require general AWS knowledge or familiarity. However, most topics are explained with sufficient background if you haven't used the particular service before.

[

## Costs

](https://catalog.workshops.aws/certificatemanager/en-US/introduction#costs)

If you are running this workshop at an official AWS event, you'll be using temporary accounts and costs will not be incurred by you. However, if you are running this by yourself, costs of all resources used will be incurred. This included EKS, AWS Private CA, IoT Core, and ALB.


# ...on your own



### Running the workshop on your own



Important

Only complete this section if you are running the workshop on your own. If you are at an AWS hosted event (such as re:Invent, re:Inforce, Immersion Day, etc), go to [Start the workshop at an AWS event](https://catalog.workshops.aws/certificatemanager/en-US/gettingstarted/aws_event/).

-   Log into your desired AWS account
    
-   Verify that you're in the desired region.
    
    -   Please use a AWS region in which AWS Cloud9, Amazon Elastic Kubernetes Service, and AWS Certificate Manager(ACM) service are [available](https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services/) 
-   Download the CloudFormation template by right clicking this link: [Security Admin CloudFormation Stack](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloudformation_templates/security_admin.yaml) and save link as _security-admin.yaml_
    
-   Upload and launch the CloudFormation stack in the AWS account that you are logged into. If you are not familiar with this, follow [instructions here](https://catalog.workshops.aws/certificatemanager/en-US/gettingstarted/self-paced/security_admin_cf_instructions)
    
    -   **Note:** the CloudFormation stack may take around 20 minutes to deploy the workshop environment, which includes the infrastructure for the [Internal Application Exercise](https://catalog.workshops.aws/certificatemanager/en-US/httpsexercise) and the [Protecting Data in Transit on EKS](https://catalog.workshops.aws/certificatemanager/en-US/eksexercise) sections

To avoid any permissions issues in your account, make sure that you have administrator access. Also, S3 block public access needs to be disabled for the CRL S3 buckets so that these buckets are accessible by the TLS client.



# ...at an AWS event



### Running the workshop at an AWS Event



Important

Only complete this section if you are at an AWS hosted event (such as re:Invent, re:Inforce, Immersion Day, or any other event hosted by an AWS employee). If you are running the workshop on your own, go to: [Start the workshop on your own](https://catalog.workshops.aws/certificatemanager/en-US/gettingstarted/self_paced/).



### Login to AWS Workshop Portal


This workshop creates an AWS account and associated resources needed. You will need the **Participant Hash** provided upon entry, and your email address to track your unique session.

Connect to the portal by clicking the button or browsing to [https://dashboard.eventengine.run/](https://dashboard.eventengine.run/) . The following screen shows up.

Enter the provided hash in the text box. The button on the bottom right corner changes to **Accept Terms & Login**. Click on that button to continue. ![Event Engine](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/event_engine/event-engine-initial-screen.png)

Choose either **Email One-Time Password (OTP)** or **Login with Amazon** ![Event Engine Dashboard](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/event_engine/eventenginelogin.png)

Click on **AWS Console** on dashboard. ![Event Engine Dashboard](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/event_engine/event-engine-dashboard.png)

Take the defaults and click on **Open AWS Console**. This will open AWS Console in a new browser tab. ![Event Engine AWS Console](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/event_engine/event-engine-aws-console.png)

---

Important

While the above screenshot shows the AWS Region as us-west-2, you should use the Region specified by your AWS workshop facilitator. If they did not specify which Region you should be operating in during this workshop, please ask the nearest AWS workshop facilitator.


### Next Steps

Once you have completed the steps above, you can head straight to

[CA Hierarchy Setup](https://catalog.workshops.aws/certificatemanager/en-US/hierarchysetup)

步骤 1:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/securityadmin_cf_instructions.png)

步骤 2:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/search_step.png)

步骤 3:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/upload_step.png)

步骤 4:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/namestack_step.png)

步骤 5:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/scroll_step.png)

步骤 6:

![CF Instructions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cf_instructions/createstack_step.png)





# CA Hierarchy Setup


### Assume IAM Role called **CaAdminRole**



-   Assume the role named **CaAdminRole** by using switch role on the AWS console in the AWS account that you are currently logged into
    
-   This role has permissions that a Certificate Authority administrator will need for CA administration. As a CA administrator, you will be responsible for creating a root and subordinate certificate authority
    
-   If you are not familiar with switching roles, follow this tutorial if needed: [Assume Role in Console](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/SwitchRole.pdf) 
    

Assuming role not working?


### Create Private CA hierarchy


-   You can create Private Root and Subordinate Certificate Authorities either manually or through CloudFormation template.
-   Please choose the manual _Private CA Creation_ or the _Quick Deploy_ option but _don't do both_.

We highly recommend that if you've never used AWS Private CA before, you use the manual instructions.

[Manual CA Hierarchy Setup](https://catalog.workshops.aws/certificatemanager/en-US/hierarchysetup/manualcahierarchy)

Want to skip manual creation steps?

Reminder that this path is only recommended if you have experience with AWS Private CA hierarchy creation. You will also be missing learning quizzes if you choose this route. Skipping manual creation will enable you to more quickly move to advanced sections. If you wish to proceed, click here: [CloudFormation deployment of CA hierarchy](https://catalog.workshops.aws/certificatemanager/en-US/hierarchysetup/autocahierarchy)








## Manually Create Private CA Hierarchy


We'll begin by creating a new CA Hierarchy: a Root CA to act as the root of trust, and a Subordinate CA that can be used to issue end-entity certificates throughout the rest of the workshop. You can find [video instructions to create a private CA here](https://www.youtube.com/watch?v=pKymN_ICpv8) .



### Create a Root CA

1.  Navigate to AWS Certificate Manager service in the AWS Console
    
2.  Expand the sidebar on the left hand side of the console, then click **AWS Private CA**
    
3.  Select **Create A Private CA**
    

---

![Create Private CA](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/acm_welcome.png)

---

4.  We will start by creating a Root CA, so leave **Root** selected under CA type options

---

![Root CA Type](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/ca_type.png)

---

5.  For **Subject distinguished name options**, enter values of your choosing. Note that **you must enter at least one name** on this page, but all fields are optional and _not all of them need to be completed._
    -   We recommend at least entering a Common Name (CN) to easily identify the CA as Root or Subordinate.

---

![Root CA Parameters](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subject_distinguished_names.png)

---

6.  Next, scroll down to configure the **Key algorithm options**. This section allows us to select a key algorithm for our CA. For this workshop, we'll use the default: _RSA 2048_.
    -   Most customers will use RSA 2048, but some customers may choose to use a longer key length (e.g. RSA 4096) for increased security
    -   Some devices may have limited processing power, and need to utilize Elliptic Curve Cryptography (ECC) certificates: ECC requires less computing power than RSA for cryptographic operations. ECDSA is often used for Internet of Things (IoT) devices.

---

![Root CA Key Algo](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/key_algo_options.png)

---

7.  Now we need to configure the revocation settings for our new Root CA. Select the checkbox for _Activate CRL Distribution_, and uncheck the _create S3 bucket_ option. Click the _Browse S3_ button and select the S3 bucket that begins with: _acm-private-ca-crl_ from the dropdown menu
    -   This bucket was pre-created in your account as part of workshop set up

---

![Root CA Revocation](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cert_revo_options.png)

---

8.  Add tags to identify the CA _(optional)_
    
9.  Under **CA permissions options** we can authorize ACM to automatically renew certificates issued by this CA. As we will not be issuing certificates directly from the Root CA, we can uncheck this box
    

---

![Root CA ACM Permissions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/ca_permissions_options.png)

---

10.  In the **Pricing** section, check the box to indicate that you agree. Now click **Create CA**
    
11.  As a final step, we need to install the CA certificate in our newly created Root CA. On your newly created CA, click the _Actions_ dropdown and select _Install CA Certificate_.
    

---

![Actions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/new_ca_actions.png)

---

12.  Here we can set the validity of the CA certificate, and the signature algorithm. Leave the settings as default, and click **Confirm and Install**.

---

![Root CA creation success](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/install_root_ca_cert.png)

---

12.  Behind the scenes, AWS Private CA creates a certificate signing request (CSR) using the information we entered in the previous steps.
    
13.  That's it! We've created a Root CA that is active and ready to issue certificates. However, AWS strongly recommends against issuing certificates directly with your Root CA. To follow best practice, we will now create a subordinate CA that will be signed by the Root CA we just created.
    

![Root CA cert review and create](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/root_complete.png)

---

[

### Root CA Quiz



-   [Quiz 1](https://bit.ly/2To8mmf) 
-   [Quiz 2](https://bit.ly/2YQr7El) 


### Create a Subordinate CA


We've created our Root CA, but now we need to create a subordinate CA that can be used to issue certificates (sometimes referred to as an issuing CA).

As the Root CA is the top of our chain of trust, we only want to use it to sign subordinate CAs, to indicate this CA is trusted by our organization. If a subordinate CA is compromised, any end-entity certificates it has issued will also be compromised. But if our Root CA's private key was somehow compromised, that would mean all certificates for our entire organization are now at risk.

We use certificate hierarchies to limit the _impact radius_ of a potential CA compromise. You can find [more information on CA hierarchies in this video](https://www.youtube.com/watch?v=8FB12c1lDyo) .

---

Follow the steps below to create a new subordinate CA that we will sign with the Root CA we created in the previous section:

---

1.  From the Private CA console, click on the **Create CA** button near the top right of the page

---

![Begin Creating Sub CA](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/sub_ca_begin.png)

---

2.  This time, select **Subordinate** and continue to configure the **subject distinguished names options**

---

![Parent CA Type](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subordinate_ca_type.png)

---

3.  Enter your desired **Subject distinguished name options**. Note that **you must enter at least one name** on this page, but all fields are optional and _not all of them need to be completed._
    -   We recommend at least entering a Common Name (CN) to easily identify the CA as Root or Subordinate.

---

![Subordinate CA Type](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subordinate_ca_parameters.png)

---

4.  Under **Key Algorithim Options** use the default, RSA 2048
    
5.  Next we need to configure our revocation method, following the same process as the Root CA creation section
    

---

![Subordinate CA Revocation](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subordinate_revocation_options.png)

---

6.  Add tags to identify the CA _(optional)_

While tags are not mandatory, they can be useful for tracking costs between parts of your organization, or they can be used to facilitate attribute-based access control (ABAC) via IAM policy.

7.  Unlike the Root CA creation section, we will leave _Authorize ACM access to renew certificates requested by this account_ checked for the Subordinate CA. This functionality can greatly reduce operational overhead by offloading certificate renewal processes to ACM vs. performing manual renewals

---

![Subordinate CA ACM Permissions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subord_perms_options.png)

---

8.  Check the box under the **Pricing** section to indicate we agree. Now click **Create CA**
    
9.  We will again need to install a CA certificate. This time, instead of a self-signed Root CA certificate, we will be using the Root CA certificate created in the previous section to sign this new subordinate CA. This will add it to our CA hierarchy and our _chain of trust_.
    
10.  On the following page, select the _Actions_ dropdown menu and _install CA Certificate_
    

---

![Subordinate Actions](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subordinate_actions_install.png)

---

11.  On this page, we need to select _AWS Private CA_ so we can pick the Root CA we created earlier

-   If you wanted to sign this new subordinate CA using a Root or Intermediate CA certificate you host on-premises, you could select _External private CA_

---

![Subordinate Cert Parent CA Type](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/install_subordinate_cert.png)

---

11.  Now we must select our Root CA as the _parent private CA_ from the dropdown menu. Leave the Validity, algorithm, and path length options on the default settings.
    -   After you select a CA from the dropdown, the common name will be populated: make sure it matches the common name of the Root CA you created in the previous section
    -   **Validity** specifies the length of time until the CA certificate expires
    -   **Path length** indicates the number of certificate authority branches/tiers that can be signed by this CA. For instance, if we were creating a 3-tier CA hierarchy with Root, Intermediate, and Issuing CAs, we would make this path length 1: this would allow this subordinate CA to act as an intermediate CA and sign CAs that are one "tier" below it in the CA hierarchy. You can find more information on [CA hierarchy and path length constraints here](https://docs.aws.amazon.com/acm-pca/latest/userguide/ca-hierarchy.html) .

---

![Subordinate Cert Parent CA Options](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/parent_ca_details.png)

[

## ![Subordinate Cert Validity](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/subordinate_ca_validity.png)


A path length of 1 does **not** mean that only one CA can be signed using the CA certificate. You could sign 5 or more issuing CAs using a CA certificate with a path length of 1, but you could **not** sign another CA below the issuing CA "tier." You can use the path length constraint to prevent other certificate authorities from signing new CAs.

12.  Lastly, we will review our selections and click **Confirm and Install**

---

13.  Great! We've created a new Subordinate CA and signed it with our Root CA, adding it to our _chain of trust_. You should see (at least) two certificate authorities in the AWS Private CA console, as shown below.

---

![Subordinate CA Creation Complete](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/sub_complete.png)

---

[

### Subordinate CA Quiz


-   [Quiz 3](https://bit.ly/2KqPgcm) 
-   [Quiz 4](https://bit.ly/2YWdJOW) 

[

### Overall CA Quiz

-   [Quiz 5](https://bit.ly/2yQ5IML) 

How should I use these quizzes?

[

### Next Steps


Once you have completed the steps above, you can head straight to the first exercise:

[Internal App Exercise](https://catalog.workshops.aws/certificatemanager/en-US/httpsexercise)



# Quick Deploy



### Automatically Create Private CA Hierarchy


Important

Only complete this section if you skipped _Private CA Creation_ section. **Do not complete both**.

-   Download and launch the CloudFormation template by right clicking this button [PCA Hierarchy Quick Deploy CloudFormation Stack](https://static.us-east-1.prod.workshops.aws/public/10d11255-d416-4508-927a-fad987ae5992/static/cloudformation_templates/pca_hierarchy.yaml) and save link as _pca-hierarchy.yaml_
    
-   Upload and launch the CloudFormation stack in the AWS account that you are logged into. If you are not familiar with this, follow instructions here: [Deploy CloudFormation Stack Instructions](https://github.com/aws-samples/data-protection/blob/master/usecase-9/img/CAAdminSteps-1.pdf)