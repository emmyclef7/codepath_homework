Background

A honeypot is a decoy application, server, or other networked resource that intentionally exposes insecure features which, when exploited by an attacker, will reveal information about the methods, tools, and possibly even the identity of that attacker. Honeypots are commonly used by security researchers to understand the threat landscape facing developers and system administrators, collecting data that might include:

1) Information about sources of malicious network traffic such as IP addresses, geographic origin, targeted ports, etc.
2) Information used to harden resources against email spammers
3) Malware samples
4) DB vulnerabilities such as SQLI techniques

There are two broad categories of honeypots:

1) Low-interaction honeypots provide simulations of target resources, typically using emulation or virtualization, to reduce resource consumption, simplify configuration and deployment, and provide solid containment features
2) High-interaction honeypots expose non-simulated target resources in a way that more closely imitates a production environment to attract more sophisticated attackers and understand more complicated exploitation routes

For example, a low-interaction honeypot might emulate a server capable of accepting SSH connections through a combination of exposed ports and decoy responses, whereas the high-interaction version would feature an actual SSH server possibly misconfigured in some way that makes it vulnerable. In the low-interaction example, attempted exploitation would quickly lead to a dead end for the attacker, perhaps revealing only an IP address and a few attempted commands to the honeypot's maintainer. In the high-interaction example, the attacker would potentially be able to compromise the server, wasting more time and giving away more information about his or her goals


Overview

In this assignment, you will stand up a basic honeypot and demonstrate its effectiveness at detecting and/or collecting data about an attack. Guided instructions for doing this using specific software are provided below, but you are free to take any approach you wish that demonstrates the following basic principles:

Successful configuration and deployment of a network-accessible honeypot server with two primary features:
1) An attack surface that is vulnerable or exposed in some way to network-based attacks
2) A network security feature such as an IDS configured to detect and log such attacks
3) Illustration of at least one attack against the honeypot that can be detected or logged in a way that captures information about the attack or the attacker


Walkthrough

Keeping in mind that there are many ways one could fulfill the above requirements, in this section, we will walkthrough a basic honeypot deployment using a well-supported open source honeypot: Modern Honey Network (MHN). MHN's architecture is modular and extensible and comes with many options for deploying different types of honey pots. In MHN architecture, there is a single admin VM which is used to deploy, manage and collect information from the honeypots, which are deployed as separate VMs. Thus to run MHN, we'll need to setup at least two VMs: the single Admin VM and at least one Honeypot VM.


Milestone 0: To the Cloud!

To complete this assignment, you'll need access to a cloud hosting provider to provision the VMs. Many providers offer time-limited free trial accounts with resource limitations, and you should easily be able to complete the requirements for this assignment within these limitations -- though you may need to ensure you cleanup before your trial period expires. The setup we'll walkthrough below has been tested to work with micro-sized VMs with < 1GB memory and < 10GB disk space, so you may often be able to use the smallest possible option when provisioning VMs. All servers in this setup use Ubuntu 14.04 (trusty) -- and most cloud providers offer this as an option. Note that Ubuntu 16.04 and 17.04 will not work.

You can use any cloud provider to which you already have access or that offers a free trial, though you'll need to be familiar with its usage and / or limitations. If you're not sure where to start, we recommend Google Cloud Platform's Free Tier, and while we'll provide general guidelines that should work with most cloud providers, the instructions below will also show insets labeled GCP Users with commands and settings specific to Google Cloud Platform. If you are confident about working with an alternate cloud provider such as AWS feel free to adapt the below instructions accordingly. If this is your first foray into the world of cloud computing, consider starting a GCP trial so you can follow the more specific instructions below.

So to get started, make sure you have authenticated access to your cloud provider. You can provision the VMs any way you like, but you will need to be able to access the VMs via SSH.


GCP Users

To begin, download and install the GCP SDK on your local machine and initialize it such that you can use gcloud from the command line. Be sure to set a default region and zone (these instructions were tested against the us-central1 region and us-central1-c zone).

    $ gcloud config set compute/region us-central1
    $ gcloud config set compute/zone us-central1-c

Once you've initialized, you should be able to run gcloud config list and see the project, region, and zone configured in the output.


Milestone 1: Create MHN Admin VM

Start by creating the MHN Admin VM via your cloud provider. The VM needs to have an internet-facing IP and accessible to you via ssh (or a similar protocol). As specified above, you can use a small or micro-sized VM for this, but the following attributes are required:

1) Ubuntu 14.04 (trusty)
2) HTTP traffic allowed (port 80)
3) TCP ports 3000 and 10000 need to be open to allow incoming (aka 'ingress') traffic That last requirement is generally the only one that may require a specific firewall rule to configure properly, because those ports are non-standard and specific to MHN. Some cloud providers may require you to create the firewall rules separately and then apply them to the VM. Either way, make sure when you create the VM that you can access it via SSH.

<img width="938" alt="firewall" src="https://user-images.githubusercontent.com/109797939/201251858-7a7a6959-cc9b-4138-80fc-272eab511016.PNG">


GCP Users

First, create a firewall rule to allow ingress traffic on TCP ports 3000 and 10000. The following command can be used to do this via the command line:

gcloud compute firewall-rules list

    gcloud compute firewall-rules create http \
        --allow tcp:80 \
        --description="Allow HTTP from Anywhere" \
        --direction ingress \
        --target-tags="mhn-admin"
    
    gcloud compute firewall-rules create honeymap \
        --allow tcp:3000 \
        --description="Allow HoneyMap Feature from Anywhere" \
        --direction ingress \
        --target-tags="mhn-admin"

    gcloud compute firewall-rules create hpfeeds \
        --allow tcp:10000 \
        --description="Allow HPFeeds from Anywhere" \
        --direction ingress \
        --target-tags="mhn-admin"

    gcloud compute firewall-rules create wideopen \
        --description="Allow TCP and UDP from Anywhere" \
        --direction ingress \
        --priority=1000 \
        --network=default \
        --action=allow \
        --rules=tcp,udp \
        --source-ranges=0.0.0.0/0 \
        --target-tags="honeypot"
    
This will prompt you to install the beta SDK; after doing so, the command should complete successfully. You can verify the mhn-allow-admin rule was created via the browser by looking at the VPC Network Firewall Ingress Rules. Next, create the VM itself, which we'll call mhn-admin:

    gcloud compute instances create "mhn-admin" \
        --machine-type "n1-standard-1" \
        --subnet "default" \
        --maintenance-policy "MIGRATE" \
        --tags "mhn-admin" \
        --image-family "ubuntu-minimal-1804-lts" \
        --image-project "ubuntu-os-cloud" \
        --boot-disk-size "10" \
        --boot-disk-type "pd-standard" \
        --boot-disk-device-name "mhn-admin"
    
Note the tags value, which controls the applicable firewall rules. The output should show both internal and external IP addresses...make note of the external IP:

<img width="742" alt="admin-honeypot" src="https://user-images.githubusercontent.com/109797939/201252430-518fcb6b-88da-4f8b-8a07-6e011ab7774a.PNG">

Finally, establish SSH access to the VM via gcloud compute ssh mhn-admin (which is similar to ssh). You'll be asked to add the fingerprint the first time you connect, and you'll see the Ubuntu welcome message and a command prompt.


Milestone 2: Install the MHN Admin Application

After having established SSH access the MHN Admin VM, the following instructions can be run on the remote VM to install the application. Note: this step may take 30-40 minutes overall. These instructions were adapted from the MHN README. First, update apt and install git:

     sudo apt-get update
     sudo apt-get install git -y

Next, clone the MHN code into /opt, change into the clone directory, and run install.sh:
 
     sudo apt update && sudo apt install git python-magic -y

     cd /opt/
     sudo git clone https://github.com/pwnlandia/mhn.git
     cd mhn/

     sudo sed -i 's/Flask-SQLAlchemy==2.3.2/Flask-SQLAlchemy==2.5.1/'g server/requirements.txt

     sudo ./install.sh
     
This will start the script running and it will take a while (approximately 20 minutes) to complete the first part, after which you'll be prompted to specify a series of values:

    Do you wish to run in Debug mode? y/n : n
    Superuser email: You can use any email -- this will be your username to login to the admin console.
    Superuser password: Choose any password -- you'll be asked to confirm.

You can accept the default values for the rest of the values, either by hitting enter or n for any y/n prompts:

    Server base url ["http://#.#.#.#"]:
    Honeymap url ["http://#.#.#.#:3000"]:
    Mail server address ["localhost"]:
    Mail server port [25]:
    Use TLS for email?: y/n n
    Use SSL for email?: y/n n
    Mail server username [""]:
    Mail server password [""]:
    Mail default sender [""]:
    Path for log file ["/var/log/mhn/mhn.log"]:

The script will churn for another 15 minutes or so, and near the end, there will be two more y/n prompts, both of which you can answer n:

    Would you like to integrate with Splunk? (y/n) n
    Would you like to install ELK? (y/n) n

Now you should be able to load the external IP in a browser and login to the admin console via the "superuser" values you chose above. Have a look around the UI to get oriented; there won't be any data available as we've not deployed any honeypots yet.

<img width="739" alt="mhn-admin" src="https://user-images.githubusercontent.com/109797939/201253288-6997ab61-3029-4b31-b6e6-7138230de84c.PNG">


Milestone 3: Create a MHN Honeypot VM

MHN supports multiple honeypots, each of which has a slightly different purpose you can read about. To start, we'll deploy Dionaea over HTTP, a honeypot used to trap malware samples.

First, create a VM for your first honeypot via your cloud provider. As before, this VM also needs to have an internet-facing IP and accessible to you via ssh (or a similar protocol), and as before, you can use a small or micro-sized VM, but it should be running Ubuntu 14.04 (trusty).

This VM will require different ports open, though which ones depend on the specific honeypot being used. To keep things simple, for this VM (and any additional honeypot VMs you create), simply allow incoming traffic from all ports and protocols. Again, this will likely require a firewall rule.

Create the VM and establish an SSH connection to it before proceeding to the next step.

    gcloud compute instances create "honeypot-1" \
        --machine-type "n1-standard-1" \
        --subnet "default" \
        --maintenance-policy "MIGRATE" \
        --tags "honeypot" \
        --image-family "ubuntu-minimal-1804-lts" \
        --image-project "ubuntu-os-cloud" \
        --boot-disk-size "10" \
        --boot-disk-type "pd-standard" \
        --boot-disk-device-name "honeypot-1"

<img width="742" alt="admin-honeypot" src="https://user-images.githubusercontent.com/109797939/201253705-093c7fe9-1da5-452a-a5dd-b6db4db946b3.PNG">

After having established SSH access the new honeypot VM, we need to install the honeypot application into the VM and wire it to connect back to the admin server. Fortunately, MHN makes this fairly straightforward. First, in the MHN admin console in your browser, click on Deploy in the top nav, and you'll be asked to select a script. Choose Ubuntu - Dionaea with HTTP, and you'll see a Deploy Command appear with a full deployment script below it. You can ignore the script, which is just for reference, but make a note of the Deploy Command, which is the one-line command you'll need to execute inside the honeypot VM you connected to in the last step.

So, copy the command from the browser page. It should start with wget and end with a unique token string. Execute this command inside the honeypot VM to install the Dionaea software. It shouldn't take more than a few minutes to complete. When it's done, click back over to the MHN admin console in your browser. From the top nav, choose Sensors >> View sensors and you should see the new honeypot listed.


Deploy Command – Ubuntu – Dionaea with HTTP

    wget "http://35.225.22.169/api/script/?text=true&script_id=2" -O deploy.sh && sudo bash deploy.sh http://35.225.22.169 sX9hZta


Milestone 5: Attack!

Now for the fun part: let's attack the honeypot to make sure it's all working. You can use nmap and pass it the IP of the honeypot VM (not the IP of the MHN admin VM):

<img width="467" alt="Nmap1" src="https://user-images.githubusercontent.com/109797939/201253984-ab55636a-d772-43a2-86f6-5d1a653f765b.PNG">

<img width="466" alt="Nmap2" src="https://user-images.githubusercontent.com/109797939/201254006-367a98fc-7a4a-4413-80b8-d7e826678397.PNG">

<img width="922" alt="Attack-report" src="https://user-images.githubusercontent.com/109797939/201254059-d27e1398-d4a5-42c7-902a-86663ca65e40.PNG">

It should show ports open...these are the services Dionaea is using to attract attackers. Switch back to the MHN Admin console in your browser, and from the top nav, choose Attacks. If everything goes well, you should see your IP address listed with several port scan records. This means the honeypot intercepted your attack.

You may, however, see other attacks as well, from other IPs. In fact, it shouldn't take long at all for this to happen. Port scans should start coming in at an alarming rate, from all over the world, and even with only a single honeypot deployed, MHN will start collecting lots of data. Welcome to the hostile territory that is the Internet.

<img width="508" alt="Attack-Summary" src="https://user-images.githubusercontent.com/109797939/201256130-5180811e-2440-41c3-9787-250185a82031.PNG">

Encountered problems viewing the attacks report even though it was visible on the map. Had to research and restart Mnemosyne in order to make it visible but for some reason after restarting it stopped showing the attacks on the map

Commands to restart Mnemosyne:

    Admin@mhn-admin:~$ cd /opt/mnemosyne/cd
    Admin@mhn-admin:~$ sudo supervisorctl restart mnemosyne
    Admin@mhn-admin:~$ sudo supervisorctl status

Map showing where attacks are coming from:

<img width="804" alt="Map" src="https://user-images.githubusercontent.com/109797939/201256330-33c5cb75-78f6-44fd-b748-3dddc06b7024.PNG">


Exporting Data

The submission for this assignment will require an export of the data collected from your honeypot(s). You'll want to leave the honeypots up and running a while -- the longer the better -- and then export the data at the end of the assignment and download it to your host machine.

GCP Users

You can do the export directly from your local machine in two steps, so run the following commands on your local machine. 

<img width="440" alt="json1" src="https://user-images.githubusercontent.com/109797939/201256412-f7f4ba37-4ebc-4eda-ba9a-a11df5cf8df9.PNG">

<img width="469" alt="json2" src="https://user-images.githubusercontent.com/109797939/201256458-34e2b5c4-17e7-4485-b482-0f193898a639.PNG">

The session.json file should be downloaded successfully.

See session.json file within this repository.


Cleanup

When the assignment is complete, you'll most likely want to remove the VMs created as part of the honeypot network to avoid costs incurred once your trial expires.
NOTE

This project took an extensive amount of time to complete (30 hours). It could have been finished in half the time but the instructions did not address many of the problems that I encountered while doing this project.



