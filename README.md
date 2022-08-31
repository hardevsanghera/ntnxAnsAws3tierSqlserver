# ntnxAnsAws3tier
<p>Ansible playbooks to deploy 3-tier Tasks Laravel web app with MySQL dbserver (on Nutanix AHV) and nginx web servers on AWS.</p>
<p>The main playbook has copious comments: ntnxawsplay.yaml</p>

<h2>Application Architecture</h2>
<p>This 3 tier application, a webapp, is deployed, using Ansible, with MySQL as the back-end database.  Two nginx servers make up the middle layer and the front-end loadbalancer is implemened using HAProxy.  The latter two layers are deployed onto AWS and the dtabase server is deployed onto a Nutanix AHV cluster.  The dtabase layer and the midle webserver layer communicate over an ssh tunnel - this means there's no need for an AWS site-site vpn and one the app has been deployed any user anywhere from any device with a browser can enter the public IP address of the loadbalancer and get to the Task Manager webapp.</p>
<img src="images/arch-ansible-small.jpeg" 
     width="500" 
     height="auto" /> 
<h2>Application UI</h2>
<img src="images/taskappiphone-small2.jpeg" 
     width="200" 
     height="auto" />

<h2>Pre-requisites</h2>
<p>I used an Ubuntu 20.04.1 workstation VM running under VirtualBox.</p>
<ol>
     <li>Ansible core 2.13.2</li>
     <li>Nutanix Ansible Module: https://github.com/nutanix/nutanix.ansible - great blog walk-thru: https://www.nutanix.dev/2022/08/05/getting-started-with-the-nutanix-ansible-module/</li>
     <li>AWS Account with valid API key and secret key, if you can run the aws cli then you rshould be good with the permissions you have</li>
     <li>AWS VPC including subnet, key pair (pem file) and inbound security group rules - see the comments in the playbook.</li>
     <li>Nutanix AHV based cluster manged by Prism Central, with credentials</li>
     <li>CentOS 7 AHV disk image, from here: http://download.nutanix.com/Calm/CentOS-7-x86_64-1908.qcow2 - the getImageplay.yaml Ansible playbook will fetch the image for you - edit it first.
</ol>
<h1>How to install and get the webapp working</h1>
<ol>
     <li>verify pre-reqs above</li>
     <li>git clone this repo</li>
     <li>Optional: edit getImageplay.yaml to reflect your Prism Central(PC)</li>
     <li>Optional: $ ansible-playbook getImageplay.yaml - Or you can use the PC UI to upload the image as CentOS7.qcow2 from the URI above.</li>
     <li>edit ntnxawsplay.yaml to reflect your PC and AWS VPC</li>
     <li>copy your ec2 key pair file (something.pem) to the repo folder where all the other files are</li>
     <li>$ ansible-playbook ntnxawsplay.yaml</li>
</ol>
<p>The last task to be run will print out the public IP addresses of the loadbalancer (HAProxy) and the two webservers.  Point your browser to the IP address of the loadbalancer and you will be routed through to the Task Manager webapp.</p>
<h1>Timings</h1>
On average complete deployment (not including the image upload) for the main ntnxawsplay.yaml (ie. the whole 3-tier application and componenets) takes about 20-30 minutes - sometimes longer.  This is because the VMs have to install packages and updates as well as perform the installtion and customization of the application.
<h1>Issues and Observations</h1>
<ul>
     <li>The ssh tunnels between the webservers and the datbase server will drop after about 2 hours - beware if demoing, advise setup maybe 45 minutes before needed.</li>
     <li>Timing:  There are "pause" tasks implemenetd in the playbook as sometimes the VMs have not quite customized or other reasons.  These should be long enough but you may need to vary them sometimes.</li>
     <li>"Unable to connect" message - on occasion the playbook task trying to connect to any of the VMs will error "could not connect" or similar message.  I advise simply to delete everything created so far and re-running the playbook.</li>
     <li>SOmetimes I got "AWS was not able to validate the provided access credentials" when running a palybook - check the time on your workstation - if it's out by only a few minutes then AWS will not accept your credentials.  Set the current time on your workstation to fix.  </li>
</ul>
