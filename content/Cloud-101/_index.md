---
title: "Cloud 101"
linkTitle: "Cloud 101"
weight: 1
#archetype: "chapter"
---

## Lab Overview
The goal of this lab is to, as quickly as possible, get a simple web site running and exposed to the internet.  This website will require some compute resources to run on, exposure to the Internet (inboud/outboud) and permissions to access a centrally managed database.

Application Diagram

{{< figure src="img/app-diagram.png" >}}

To achieve your goal you will use the public cloud to create resources, download code, grant permissions and finally expose your website to the entire world.  All this will be achieved using the Amazon Web Services (AWS) web console,a web browser and a few simple commands.

Good luck!
## Log into Qwiklabs
Your first task will be to create an new AWS cloud account using Qwiklabs.

[Fortinet's Qwiklabs](https://fortinet.qwiklabs.com/classrooms/90)

Use the email/password provided by your lab teachers to log in to Qwiklabs.

{{< figure src="img/qwiklabs-login.png" >}}
## Select the course
After you log into Qwiklabs select the in progress course 'Public Cloud 104 CNAPP'.

{{< figure src="img/qwiklabs-select-course.png" >}}

## Select the lab
Next select the 'Lab 1: Create AWS Cloud Account'.

{{< figure src="img/qwiklabs-select-lab.png" >}}

## Follow the directions in the Qwiklabs lab
Return here after you have access to your brand new AWS cloud account.

## Create an Virtual Server
Now that you have access to the AWS Console let's get working!

To get your website working you will need a server where you can run the code.  In a traditional on premise data center you would request a server from your operations team.  In the public cloud you can create your own resources, include servers, without having to ask anyone.

In AWS servers can be created using their `EC2` service. Use the search box and type `EC2` and click on the top returned result.

{{< figure src="img/aws-console-us-east-1.png" >}}

The EC2 service enables creating virtual servers in many different regions around the world. By default, you will be dropped into `us-east-1` region which is located in the United States, North Virginia.

Find `Instances` in the left hand navigation (Below the Heading of "Instance") and click on it to view your currently running virtual servers. At the moment you should have none running.

Click on `Launch Instance`

{{< figure src="img/ec2-1.png" >}}

Give you new virtual server a name you will remember, keep it safe for work!
Next choose Ubuntu as the operating system you want running in your new virtual server.

{{< figure src="img/ec2-2.png" >}}

Now we need configure what credentials we will use to access this virtual server once it is up and running.
Select `FortiCNAPPKey` from the drop-down.  This key was created by Qwkilabs and can be used within AWS or from you local machine to access your virtual server.

{{< figure src="img/ec2-3.png" >}}

Finally click on `Launch Instance` to create you brand new virtual server.

{{< figure src="img/launch.png" >}}

Phew! You now have a brand new virtual server up and running in EC2.  Click on the ID for you new virtual server to view more details.

{{< figure src="img/aws-ec2-new-server.png" >}}
## Access your Virtual Server
Next you need to gain access to your new virtual server so you can make it actually do something useful.  Developers often connect to servers using a secure shell (SSH).  AWS offers a simple way to do this called `Instance Connect`.  Instance Connect will allow you to gain a connection to your new virtual server using your browser.  After gaining access you will be able to run commands on the server.

Right-click on the instance name of your new instance and choose `Connect`.

{{< figure src="img/aws-4.png" >}}

On the Connect to instance window, keep the default option of `EC2 Instance Connect` and click on `Connect`

{{< figure src="img/aws-5.png" >}}

If successful, you should see a terminal like interface within your AWS Console.

{{< figure src="img/aws-6.png" >}}
## Ease into the Terminal Experience
Let's run a few simple commands to get comfortable on the terminal.  Copy each command below and paste it into the terminal interface.

First let's run the command that Prints the current Working Directory we are in: pwd
```bash
pwd
```
{{< figure src="img/cli-pwd.png" >}}

When we logged into our virtual server we were dropped into the home directory of the ubuntu user.

Next let's list all the files or directories in the current working directory: ls -lah
```bash
ls -a
```
{{< figure src="img/cli-ls-a.png" >}}

In addition to runing the 'ls' command you passed a single argument 'a'. The 'a' modifier tells the 'ls' command to list all of the files or directories, even hidden ones.

Finally let's see how we can learn more about any Linux command.  Here we will use the 'man' command.  The 'man' command will explain what a command does and what possible arguments the command will accept.
```bash
man ls
```
{{< figure src="img/cli-man-ls.png" >}}

Notice that we can see the an explanation of the '-a' argument we passed into 'ls'. The argument directs 'ls' to not ignore entries starting with '.'.  Any file or directory in Linux starting with '.' is normally hidden from listing.

To exit the 'man' command hit the 'q' key.

For bonus points try running the 'man' command to explain the what the 'man' command does.
```bash
man man
```
## Download the website code
The code for your new website is stored in git repository hosted on GitHub. Git/GitHub allow developers to collaborate together and share code.  Almost no modern company develops all of their own code. Developers rely on using many other projects to save time and effort while delivering business value.

The command below will download the code to your virtual server

```bash
git clone https://github.com/forticnapp-cloud-lab/hello-world.git
```

You can confirm that code is on your server by listing all the file us the `ls` command.

```bash
ls hello-world
```
## Make the site your own
Before you start up your new website let's make a few simple changes. The default message is `Hello World!`.  The is kind of boring.  Let's update that message to something a bit more fun.

You will be using a program called `nano` to make this change. Below is the command to start `nano` opening the index.js file that runs your website.

```bash
nano ~/hello-world/index.js
```

{{< figure src="img/aws-ec2-nano-hello-world.png" >}}

Nano tries to be as user friendly as it can be.  You can use your arrow keys to move around, use delete to remove text and type to add text.

Let's make a change to the website to make it your own. Find `Hello World!` and replace the text with your own message. You can customize your website even more by replacing `IMAGE_URL` with a URL for an work safe image you can find on the world wide web.

Once you are done making changes use the control key and `o` key, followed by enter to confirm the change.  Finally control 'x' to exit `nano`.
## Install node runtime
Before we can even start up your website you will need to get the Node runtime setup.  Node is a more modern version of the browse language JavaScript.  Many modern websites and services use Node.

The command below will use the Ubuntu built in package management application (apt) to download NodeJs (the Node runtime) and npm (Node's own package manager).

```bash
sudo apt update && sudo apt install -y nodejs npm
```

Notice how many files are installed running just these few commands. Hopefully there aren't any negative side effects of making all these changes.
## Install your website dependencies
As we mentioned all modern software is built on top of other software.  Your simple website is no exception.  In Node these software dependencies are listed in a file called `package.json`.

To see your dependencies run this command. The `cat` command will output the content of a file.

```bash
cat hello-world/package.json
```

{{< figure src="img/aws-ec2-cat-package.png" >}}

Before you can start your website you will need to install these dependencies.  Luckily you already installed the Node package manager `npm`. Run the command below to change directory and install the needed Node packages.

```bash
cd ~/hello-world
npm install
```

{{< figure src="img/aws-ec2-npm-install.png" >}}

Notice that we only had 4 direct dependencies in our `package.json` file, but we actually added 229 packages. This is because our direct dependencies have their own dependencies, which in turn might yet more dependencies.  Software is built on top of software. Turtles all the way down.

Even more concerning is the warning that there are 8 vulnerabilities found, including 5 of high severity.

Well no time to worry about all of that, you have a website to get up and running!
## Start up your website
To keep your website up and running you will use the `pm2` binary installed in the step above using npm.  The job of `pm2` is to ensure that your website stays up and running even if the virtual server gets restarted or your website crashes.

```bash
./node_modules/pm2/bin/pm2 start index.js
```

{{< figure src="img/ec2-node-started.png" >}}
## Access your brand-new website
Go back to the AWS EC2 Service, click on your instance. You will find the public IP address of that instance and copy it.

{{< figure src="img/ec2-get-public-ip.png" >}}

Note: use the copy link and not the 'open address' link

You will need to add the port the website is running on `3000`.  Let's hope that port is actually open, otherwise we might not be able to access our new website.

`http://<Your-IP>:3000`

> [!info] Take Notice
> Ensure that you include 'http://' as modern browsers will try and force you onto 'https://', which listens on a different port (433).

## Hello is anyone listening?
You might have noticed that when you visited the URL of your newly launched website nothing happened, your browser tab just spun and spun. AWS EC2 be default only allows inbound network connections on port 22, the default port of SSH.  Since your website is not running on port 22 all other inbound traffic was blocked. Next task is to open up some ports to allow inbound traffic to reach your virtual server.

AWS EC2 manages network access via `Security Groups`. Return to AWS EC2, find you EC2 instance, switch to the `Security` tab and click on your security group name.

{{< figure src="img/aws-ec2-open-security-group.png" >}}

Ensure you are on the `Inbound Rules` tab and click on `Edit inbound rules`.

{{< figure src="img/aws-ec2-lauch-inbound-rules.png" >}}

You will need click `Add rule` to create a new inbound rule. Then specify the `Protocol` as `TCP`, `Port range` of `0-65000` and ensure anyone on the internet can access by specifying a `Source` of `0.0.0.0/0`.

Notice that we opened up way more ports than we need and AWS warns us about allowing all inbound IP addresses. These are just the type of decisions that are now entrusted to anyone with a cloud account.

Anyway... click `Save rules`, let's get this web site up!

{{< figure src="img/aws-ec2-edit-inbound-rules.png" >}}

Refresh the URl of your website to see if we can get traffic flowing.
## Wait that doesn't look right
Your new website isn't showing your fancy message. Rather it is complaining about an IAM role not being attached.

{{< figure src="img/website-no-iam-role.png" >}}

As you might recall from the very beginning we mentioned that your website would need access to a centrally managed database. Access to cloud resources, like databases, is allowed through Identity and Access Management (IAM). IAM has conecpts like roles, policies and permissions that control what can and can't be accesssed by users and cloud resources.

By default when you create a new EC2 instance there is no IAM role attached.  This means that your virtual machine can not access any other cloud resources at the moment.

{{< figure src="img/aws-ec2-no-role-defined.png" >}}
## Create a new IAM role
To attach a new IAM role to your virtual server click on `Action`, then `Security` and finally `Modify IAM role`.

{{< figure src="img/aws-ec2-modify-role.png" >}}

Before you can attach a role you will have to create a new on.  Click on `Create new IAM role` to open a new window in the IAM service page.

{{< figure src="img/aws-attach-role-before-new-role.png" >}}

On the IAM Role page you will find a list of all the current roles.  You want a new role that can be attached to your virtual machine so click on `Create role`.

{{< figure src="img/aws-iam-role-page.png" >}}

There are a few type of roles you can create select `AWS service` and then in the `Service or use case` drop down select `EC2`.  This will allow your to attach this new role to an EC2 virtual server.

{{< figure src="img/aws-iam-create-ec2-role.png" >}}

Now comes the time to choose which permissions to grant.  Notice that there are over a thousand possible permissions policies you could grant.  Rather than go over all of these trying to find the right one, we will take the simple path and grant `AdministratorAccess`. Choosing the admin role grants more permissions than you need, but ensures that the database access will work. Once again these are just the types of decisions that anyone with a cloud account face everyday.

{{< figure src="img/aws-iam-grant-admin.png" >}}

Finally the hardest choice you have made all day. What do you name this new role? Pick something easy to remember and click `Create role`.

{{< figure src="img/aws-iam-create-role.png" >}}

> [!info] Take Notice
> You can close the IAM role creation page, we are done here!

## Attach IAM role to your EC2 Virtual Server
Return to the previous page where you where trying to attache a role.  Click the circle icon, select your new role from the drop down and click `Update IAM role`.

{{< figure src="img/ec2-attach-new-role.png" >}}

Finally check your website again. Fingers crossed it all worked.
## Take moment and recap
Here are the high level steps you took:
<ul>
<li> created a new virtual machine
<li> cloned website code from github
<li> updated the website code
<li> installed the Node runtime
<li> installed node packages via npm
<li> started the website
<li> configured the network access
<li> create a new role and attached it to your virtual machine
<li> access your wonderful new website
</ul>
All of this happened without involving IT, development, security, management, operations.

What could go wrong?
## Take a break already
Once you complete this section of the lab, please take a break and talk amongst your peers about what you learned, we will all come back together once everyone has finished

{{< figure src="img/break-time-7298ce200c.jpg" >}}
