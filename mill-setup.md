# DuraCloud Mill Setup
The mill can be deployed (and updated) using a simple python commandline tool we created for this purpose.
It works by reading several configuration files which it then uses to create  the aws resources  on which the mill 
depends (ie queues, alarms, launch  configs, and autoscale groups).  

## Deploy the mill for the first time
1. First clone the mill deploy tool repository: 
    ```$xslt
    git clone https://github.com/duracloud/mill-deploy.git
    ```
1. Next create your mill config directory where you will keep your mill config files.
```$xslt
# create a local directory for your configs
mkdir -p mill-config/production
```
1. Then copy all the files into the above dir from the latest tagged release of mill-deploy/sample-config:

1. Edit each file, filling in the appropriate values based on the values generated in the 
[initial AWS setup](aws-setup.md).

1. Follow the instructions for deploying the mill in the mill-deploy/README.

## Deployment Notes

* When configuring the size of the Duplication instances, be aware that the duplication process requires a decent amount of memory to execute. In testing, a t2.micro instance is not enough, but a t2.medium seems to work well.

## Deploy the duplication policy editor

The Duplication Policy Editor allows the DuraCloud administrator to set the duplication policies for each space
within an account.  As a note,  we are currently in the process of changing the way the duplication policies work, 
moving from an "opt-in" model with no duplication occurring by default to an "opt-out" model.  That said, the structure
of the underlying data won't change so there will be a straightforward path for migration.

1. In order to deploy the Duplication Policy Editor,  you can copy the contents of the mill/policy-editor/src/main/webapp 
directory to a publicly available directory on your webserver.  You alternatively put these files in a public S3 bucket
that has been configured to serve as a website.  For more information,
[see this article]( http://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html).

2. The Duplication Policy Editor uses that DuraStore Javascript client for persisting Duplication Policies.  
Therefore, on order to use the Editor, you'll need to make sure that the *.duplication-policy-repo bucket that you
set up in the [AWS Setup](aws-setup.md) step, is visible as a DuraCloud space.  To achieve that,  you'll need to setup
a bootstrap duracloud account using  credentials associated with the account that create the *.duplication-policy-repo. 
Once your [DuraCloud Application](duracloud-webapp-setup.md) is deployed you should be able to follow the steps to 
[create a new account](creating-new-accounts.md), employing the account that you have already been using to host your
DuraCloud core infrastructure.

## How-Tos

### How to pause the Manifest Cleaner

Occassionally you may want to pause the activity of the manifest cleaner in order to lighten the load on the database.  The manifest cleaner (manifest-cleaner-<version>.jar) is responsible for purging manifest records that have been flagged for deletion.  When the contents of large spaces are deleted (as is the case when a  bridge snapshot completes), a large number of deletes will hit the database at the same time.  When those deleted manifest entries are purged from the manifest table,  it can result in slower performance of the manifest table indices which can in turn slow the throughput of writes to the manifest table. In turn this can exacerbate a back up of unprocessed audit messages.  To ameliorate this situation, you can temporarily suspend the manifest-cleaner until the audit queue has been consumed to a reasonable size.

1. Log into the sentinel.
2. ``cd /home/duracloud``
3. ``sudo mv manifest-cleaner-<version>.jar manifest-cleaner.jar.offline``
4.  ``sudo service manifest-cleaner stop``
5. Verify that the manifest-cleaner java process is no longer running.