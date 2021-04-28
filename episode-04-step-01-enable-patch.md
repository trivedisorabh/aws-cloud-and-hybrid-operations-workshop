# Enabling Inventory

![](media/ssm-aws-logo.png)

NOTE: You will incur charges as you go through either of these workshops, as they will exceed the [limits of AWS free tier](http://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier-limits.html).

## Table of Contents

- [Summary](#summary)
- [Instructions](#instructions)
    - [Create prerequisite resources using CloudFormation](#create-prerequisite-resources-using-cloudformation)
    - [Create a patch baseline](#create-a-patch-baseline)
    - [Create and assign a patch group](#create-and-assign-a-patch-group)
    - [Scan instances for missing updates](#scan-instances-for-missing-updates)
    - [Review patch compliance](#review-patch-compliance)
    - [Export patch compliance](#export-patch-compliance)
    - [Install missing updates](#install-missing-updates)
- [Next Section](#next-section)

## Summary

In this section you will first create prerequisite resources for AWS Systems Manager Patch Manager, Automation, and Change Manager using an AWS CloudFormation Stack. Following this step, you will create a patch baseline, scan instances for missing updates, review patch compliance, export patch results, and install missing updates.

## Instructions

### Create prerequisite resources using CloudFormation

Need to include steps on enabling Config, create S3 bucket, create Resource Data Sync, create EC2 instances, create SSM IAM role

### Create a patch baseline

Patch Manager uses **patch baselines**, which include rules for auto-approving patches within days of their release, as well as a list of approved and rejected patches. Later in this workshop we will schedule patching to occur on a regular basis using a Systems Manager Maintenance Window task. Patch Manager integrates with AWS Identity and Access Management (IAM), AWS CloudTrail, and Amazon EventBridge to provide a secure patching experience that includes event notifications and the ability to audit usage.

{{% notice warning %}} AWS does not test patches for Windows Server or Linux before making them available in Patch Manager. Also, Patch Manager doesn't support upgrading major versions of operating systems, such as Windows Server 2016 to Windows Server 2019, or SUSE Linux Enterprise Server (SLES) 12.0 to SLES 15.0. Always test patches thoroughly before deploying to production environments. This is a customer owned responsibility.
{{% /notice %}}

**To create a patch baseline**

1. Open the AWS Systems Manager console at https://console.aws.amazon.com/systems-manager/.
1. In the navigation pane, choose [**Patch Manager**](https://console.aws.amazon.com/systems-manager/patch-manager).
1. Select the **View predefined patch baselines** link under the **Configure patching** button on the upper right.

    **Note**: If you have previously configured Patch Manager, choose the **Patch baselines** tab.
    
1. Choose **Create patch baseline**.
1. On the **Create patch baseline** page, in the **Patch baseline details** section, perform the following steps:

    - For **Name**, enter ```AmazonLinux2SecAndNonSecBaseline```.
    - For **Description**, optionally enter a description, such as: ```Amazon Linux 2 patch baseline including security and non-security patches```.
    - For **Operating system**, choose **Amazon Linux 2** from the list.
    - Choose **Set this patch baseline as the default patch baseline for Amazon Linux 2 instances.**

1. In the **Approval rules** section, perform the following steps:

    - For **Rule 1**:
        - For **Product**, choose **All**.
        - For **Classification**, choose **Security** and **Bugfix**.
        - For **Severity**, choose **All**.
        - Leave the **Auto approval delay** at its default of **0 days**.
        - For **Compliance reporting - optional**, choose **Critical**.
    - Choose **Add rule**.
    - For **Rule 2**:
        - For **Product**, choose **All**.
        - Do not select options for **Classification** or **Severity**.
        - Leave the **Auto approval delay** at its default of **0 days**.
        - For **Compliance reporting - optional**, choose **Medium**.
        - Choose **Include non-security updates**.
    - **Note** If an approved patch is reported as missing, the option you select in **Compliance reporting**, such as ```Critical``` or ```Medium```, determines the severity of the compliance violation reported in System Manager **Compliance**.
    
    ![](/media/patch-create-baseline.png)

1. In the **Patch exceptions** section, perform the following steps:

    - In the **Approved patches** text box, enter ```kernel*```.
    - In the **Approved patches compliance level - optional** section, choose **High** from the drop-down list.
    - Choose **Approved patches include non-security updates**.
    
    ![](/media/patch-add-exceptions.png)

    - **Note**: For Linux operating systems, you can optionally define an [alternative patch source repository](https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-manager-how-it-works-alt-source-repository.html).

1.  Select **Create patch baseline** and you will go to the **Patch Baselines** page where the AWS provided default patch baselines are displayed. Your custom baseline can be found on the second page or choose **View details** in the banner displayed.

![](/media/patch-view-baseline.png)

### Create and assign a patch group

A [patch group](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-patch-patchgroups.html) is an optional method to organize instances for patching. For example, you can create patch groups for different operating systems (Linux or Windows), different environments (Development, Test, and Production), or different server functions (web servers, file servers, databases) and register each patch group to an appropriate patch baseline. Patch groups help ensure that you are deploying the appropriate patches, based on the associated patch baseline rules, to the correct set of instances. Patch groups can also help you avoid deploying patches before they have been adequately tested.

You create a patch group by using resource tags. Unlike other tagging scenarios across Systems Manager, a patch group must be defined with the tag key: ```Patch Group``` (tag keys are case sensitive). You can specify any value (for example, ```web-servers```) but the key must be ```Patch Group```. Additionally, a patch group can be registered with only one patch baseline for each operating system type; also an instance can only be in one patch group.

**Important:** When targeting managed instances for patch scan or install operations, you can target existing resource tags on the managed instance and do not need target the **Patch Group** key-value pair. For example, you can target ```ENV : DEV``` and patch baseline determination will be performed by the SSM Agent installed on each managed instance. For an example on targeting a common tag across multiple instances that use different baselines, see the topic [How it works](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-patch-patchgroups.html#how-it-works-patch-groups) in the Patch Manager section of the AWS Systems Manager User Guide.

1. Open the Amazon EC2 console at https://console.aws.amazon.com/ec2.
1. In the navigation pane, choose **Tags**.
1. Select **Manage Tags**.
1. Select the two EC2 instances with the tag ```Name``` and values: ```App1``` and ```App2```.
1. In the **Add Tag** section:

    - **Key:** Patch Group
    - **Value:** App

    ![](/media/ec2-tags-patch-group-app.png)

1. Select instances with **Name** ```Web1``` and ```Web2```
1. Add Tag

    - **Key:** ```Patch Group```
    - **Value:** ```Web```
    ![](/media/ec2-tags-patch-group-web.png)

1. Navigate back to [Systems Manager \> Patch Manager \> Patch Baselines](https://console.aws.amazon.com/systems-manager/patch-manager/baselines).
1. Select the second page of results and then select the Baseline you created in the previous part (```AmazonLinux2SecAndNonSecBaseline```).
1. Choose **Actions** and **Modify patch groups**.

    ![](/media/patch-modify-group.png)

1. For **Patch groups**, enter ```App```, choose **Add**, and choose **Close**.

    ![](/media/patch-add-group.png)

From here you can utilize the AWS provided Command document **AWS-RunPatchBaseline** to scan or patch your instances.

{{% notice note %}} Outside of this workshop, you can also select Configure Patching and link the Patch Baseline to the Maintenance Window (or create a new Maintenance Window), it will register the run task with the maintenance window and also register the Patch Group as a target. It utilizes the existing role AWSServiceRoleforAmazonSSM.
{{% /notice %}}

### Scan instances for missing updates

Now that we have created a patch baseline to define the criteria for the type of updates to approve, we will run a manual **Scan** operation to identify any missing updates on our managed instances.

**To run a scan operation**

1. Open the AWS Systems Manager console at https://console.aws.amazon.com/systems-manager/.
1. In the navigation pane, choose [**Patch Manager**](https://console.aws.amazon.com/systems-manager/patch-manager).
1. Choose **Patch now**.
1. For **Patch operation**, leave the default as **Scan**.
1. For **Instances to patch**, choose **Patch all instances**.
1. For **Patching log storage**, choose the S3 bucket created by the CloudFormation template. **Note**: The S3 bucket is named similar to ```ssm-command-logs-us-east-1-123456789012```.
1. Leave **Lifecycle hooks** disabled for the time being.
1. Choose **Patch now**.

    ![](/media/patch-scan-now.png)

A [State Manager association](https://docs.aws.amazon.com/systems-manager/latest/userguide/sysman-state-about.html) is then created to perform a **Scan** operation on your EC2 instances using the document ```AWS-RunPatchBaseline```. 

1. (Optional) To view the association, choose the **Association ID** link.

    ![](/media/patch-now-results.png)

1. (Optional) To view the command log details, choose the **Execution ID** link and then choose **Output** for one of the targeted managed instances. This will bring you to the corresponding Run Command output results for the **Scan** operation.
    
    - In **Step 2**, expand **Output** to view the command output details. You can then optionally choose **Amazon S3** to open the logs exported to the S3 bucket.

### Review patch compliance

After your instances have successfully completed a **Scan** or **Install** operation using Patch Manager, you can navigate back to the dashboard to review the patch compliance state of your managed instances, view recent patching operations, and view recurring patch tasks.

**To review patch compliance**

1. Open the AWS Systems Manager console at https://console.aws.amazon.com/systems-manager/.
1. In the navigation pane, choose [**Patch Manager**](https://console.aws.amazon.com/systems-manager/patch-manager).
1. On the **Dashboard** tab you can review the compliance status of the EC2 instances created.

    ![](/media/patch-dashboard.png)

### Export patch results

Patch Manager supports the ability to generate patch compliance reports for your instances and save the report in an Amazon S3 bucket of your choice, in .csv format. Then, using a tool like [Amazon QuickSight](https://docs.aws.amazon.com/quicksight/latest/user/), you can analyze the patch compliance report data. You can generate a patch compliance report for a single instance, or for all instances in your account. You can generate a one-time report on demand, or set up a schedule for reports to be created automatically. You can also specify an Amazon Simple Notification Service topic to provide notifications when a report is generated. For reference after this workshop, see [Generating CSV patch compliance reports](https://docs.aws.amazon.com/systems-manager/latest/userguide/patch-compliance-reports-to-s3.html). 

### Install missing updates



## Next Section

Click the link below to go to the next section.

[![](media/codify-runbooks.png)](/episode-01-step-02-codify-runbooks.md)