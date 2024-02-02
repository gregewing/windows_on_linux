<b>Windows on Linux </b><br>
Have you ever wanted or needed to be able to run a Windows instance on your Linux docker infrastructure without having to install another hypervisor like virtualbox or vagrant?  Well, with these docker container images, you can do exactly that.  Then you can connect to it using any suitable RDP Client and access a full windows desktop environment. 

The script in this project <code>BuildWindowsOnLinuxImage</code> is used to create the docker images described below.  To run the script, clone this project locally to your machine and run the script, specifying the vagrant box that you want.  Currently if you don't specify the image you want, windows10 will be be bundled as a default.   If you want a vagrant image with no embedded image, run the script with as follows:

You will need all the files from the project in a single folder as they are all required to build the bundled vagrant box in a docker image.  There is copious commenting in the code to describe whats going on, but the gist of it is as follows:

  Step 0 - Prepare to build, and figure out what we've been asked to build<br>
  Step 1 - Build an image with vagrant installed, ready for customisation<br>
  Step 2 - Build on that base and customise it with the required box<br>
  Step 3 - Run the image we just created and let it download the vagrant box<br>
  Step 4 - Watch to make sure docker, and vagarnt, startup completely<br>
  Step 5 - Harvest only the files we need to keep things as small as possible<br>
  Step 6 - Halt the Vagrant box cleanly, stop and remove docker container<br>
  Step 7 - Build the intended contaier image using the data captured so far<br>
  Step 8 - Remove the intermediate image used to capture the vagrant data<br>
  Step 9 - Publish the results to hub.docker.com repository<br>

The reason I wrote so many steps is so that we can build as small a docker image as possible.  When you initialise a vagrant box under normal circumstances, it will download the vagrant box, then copy it locally into a local cache.  To get a running instance of the vagrant box it is sufficient to keep only one copy of the image and symlink it to the repository where vagrant wants it to be, so thats what we do here, amonng a few other things to make things reliable and to enable simple configuration of the CPU. RAM and DISK allocated to the vagrant box.  I went with this multi-stage approach because there are other metadata files that need to be included, and it give me the opportunity to delete what is not needed in the docker container to keep things relatively small, well half the size they otherwise would be, on disk.

For this to work, Vagrant needs to make use of your CPUs Virtualisation Extensions ( e.g. Intel VT-x ). So either your physical host need to have a CPU with these extensions physically present in the silicon and also enabled in BIOS, or your VMWare or HyperV platform has to expose these to the guest OS in which you are running docker. You can check by running the :minimalist Image, if it starts without error, then you're good to go.  Or, if you prefer, by running the following command on your docker host:
````
sudo egrep -c '(vmx|svm)' /proc/cpuinfo
````
<b>Why did I make these images ?</b><br>
I did this because I needed to run windows and I did not have hardware available to install it on, and I did not want to install a hypervisor in addition to docker on my linux servers. When researching how I might achieve this, I found a very small number of very useful articles (links below) and have taken the ideas provided there and developed them further.  

The result is a set of containers that start very rapidly compared to pulling box images and provisioning vagrant each time.  On my modest 4 core 16 GB desktop I have windows server 2019 running in around 30 seconds from starting a previously deployed docker container. (Certainly faster than powering on hardware, or creating a clone in HyperV or VMWare.)

I take zero credit for creating the vagrant images provided here, they are available independently at <a href="https://app.vagrantup.com/boxes/search">https://app.vagrantup.com/boxes/search</a>

Windows variants available here:
<table><tbody>

<tr>
<td style="text-align:center"; colspan="1" col width=33%><b>Windows Version</b></td>
<td style="text-align: center;" colspan="1" col width=33%><b>Image Tag</td>
<td style="text-align: center;" colspan="1" col width=33%><b>vagrant image source</td>
</tr>

<tr>
<td style="text-align: left;">none </td>
<td style="text-align: center;">:minimalist</td>
<td style="text-align: center;">none - it just vagrant ready to run any image you want from <a href="https://app.vagrantup.com/boxes/search">https://app.vagrantup.com/boxes/search</a></td>
</tr>

<tr>
<td style="text-align: center;">Windows 10 Enterprise x64 eval</td>
<td style="text-align: center;">:peru_windows-10-enterprise-x64-eval</td>
<td style="text-align: center;"><a href="https://app.vagrantup.com/peru/boxes/windows-10-enterprise-x64-eval">peru/windows-10-enterprise-x64-eval</a></td>
</tr>

<tr><td style="text-align: center;">Windows Server 2016 Standard x64 eval</td>
<td style="text-align: center;">:peru_windows-server-2016-standard-x64-eval</td>
<td style="text-align: center;"><a href="https://app.vagrantup.com/peru/boxes/windows-server-2016-standard-x64-eval">peru/windows-server-2016-standard-x64-eval</a></td></tr>

<tr><td style="text-align: center;">Windows Server 2019 Standard x64 eval</td>
<td style="text-align: center;">:peru_windows-server-2019-standard-x64-eval</td>
<td style="text-align: center;"><a href="https://app.vagrantup.com/peru/boxes/windows-server-2019-standard-x64-eval">peru/windows-server-2019-standard-x64-eval</a></td></tr>

<tr><td style="text-align: center;">Windows Server 2022 Standard x64 eval</td>
<td style="text-align: center;">:peru_windows-server-2022-standard-x64-eval</td>
<td style="text-align: center;"><a href="https://app.vagrantup.com/peru/boxes/windows-server-2019-standard-x64-eval">peru/windows-server-2022-standard-x64-eval</a></td></tr>
</tbody></table>

<b>What I did do</b><br>
<ul style="list-style-type:circle">
  <li>built a container using Ubuntu22.04 and added Libvirt, kvm-qemu and vagrant</li>
  <li>scripted the customisation of the Vagrant config file to enable docker ENVironment variables to manage t-shirt size</li>
  <li>scripted a multi-stage build process to :</li>
    <ul style="list-style-type:disc">
    <li>build an empty (:minimalist) container image</li>
    <li>use that image to embed a pre-configured vagrant box</li>
    <li>strip out unnecessary bloat from the container to keep downloads reasonable</li>
    <li>create a cache of necessary vagrant files on the build host to speed up re-builds.</li>
    </ul>
</ul>

The ENV Variables allow an administrator to quickly define the CPU, RAM and DISK allocated to the vagrant image using docker command line or Portainer. See below for more info.




<b>About persistence</b><br>
The purpose of these images is development and testing.  They provide immutable builds as a starting point and provide no persistence.  Once you delete the docker container, all the changes you made to the vagrant image are gone.  Persistence is possible in theory, but I've not looked into it yet. Preliminary research suggests that it is best achieved by installing the vagrant-persistent-storage with <code>vagrant plugin install vagrant-persistent-storage</code>.  Most likely it would require mounting a volume to /vagrant from the host into the docker container.


<b>About resource allocation</b><br>
The best article I found that described the potential benefits from docker resource allocation to scenarios like this was written by <a href="https://medium.com/axon-technologies/installing-a-windows-virtual-machine-in-a-linux-docker-container-c78e4c3f9ba1">Abed Samhuri</a>. In my own testing I was able to have 6 parallel instances each with 4 GB RAM and 4 CPU Cores running in parallel on my 12 year old 4 core (+hyper-threading) 16 GB RAM desktop before I started noticing performance and usability issues. The 7th instance started up, but the page file was full on the host and things ground to a halt. Your mileage may vary.



<b>Getting Started</b><br>
This section talks about the :minimalist image, there is more below in relation to the other images.

The following docker run command will give you a container that you can use to run whatever vagrant image you like.  I have not tested them all, so your success may vary based on how the image was created, but basically any image for libvirt should at least start here if it starts on any other libvirt platform. If the image uses VNC for remote desktop, be sure to expose the correct ports.<br>
<b>N.B.</b> some of these options may not be suitable for all environments, given the general loosening of security)

<b>This will give you a container running Vagrant, but has no nested vagrant box image<br>
To start a container with a nested vagrant box image that starts automatically, look farther down this page.</b>
````
docker run -it --privileged \
--cgroupns host \
--name windows_on_linux \
--device=/dev/kvm \
--device=/dev/net/tun \
-e MEMORY=4096 \
-e CPU=4 \
-e DISK_SIZE=50 \
-v /sys/fs/cgroup:/sys/fs/cgroup:rw \
-p 0.0.0.0:3389:3389/tcp \
--cap-add=NET_ADMIN \
--cap-add=SYS_ADMIN \
gregewing/windows_on_linux:minimalist \
bash
````


Some of that might require some explanation, so i'll break it down.  You may want to tweak some of these to suit your environment.
<ul>
<li><code>--privileged</code> mode is required in order for the container to be able to assign resources from the host machine.</li>
<li><code>--cgroupns host</code> ensures that the host cgroup namespace is used when allocating resources to the container and to the vagrant image.</li>
<li><code>--name</code> is just what you want the image to be called, make this whatever you wish.</li>
<li><code>--device=/dev/kvm</code> exposes the host CPU virtualisation features (does not require kvm to be installed on the host)</li>
<li><code>--device=/dev/net/tun</code> exposes network tunnelling to enable access to the vagrant image.</li>
<li><code>-e MEMORY=4096</code> (optional, include it if you want to allocate a different amount of RAM to the Vagrant image.)</li>
<li><code>-e CPU=4</code> (optional, include it if you want to allocate a different amount of CPU to the Vagrant image.)</li>
<li><code>-e DISK_SIZE=50</code> (optional, include it if you want to allocate a different amount of DISK to the Vagrant image. Note: this may not have an effect of some vagrant box images)</li>
<li><code>-v /sys/fs/cgroup:/sys/fs/cgroup:rw</code> exposes read/write access to the hosts cgroup resource allocation subsystem.</li>
<li><code>-p 0.0.0.0:3389:3389/tcp</code> exposes TCP port 3389 on the host and forwards connections to it to port 3389 on the vagrant guest image (RDP Access)</li>
<li><code>--cap-add=NET_ADMIN</code> (optional, include it if something does not work.)</li>
<li><code>--cap-add=SYS_ADMIN</code> (optional, include it if something does not work.)</li>
</ul>

When the container starts up you will be presented with a command prompt in the working directory <code>/vagrant</code>.  From here you can initialise any vagrant libvirt image that you want, for example a Alpine Linux with the example commands below:

````
vagrant init generic/alpine318
vagrant up
````
The application of the ENV Variables is done automatically by the startup script in the docker container, but this requires that the file <code>/vagrant/Vagrantfile</code> exists before the container starts, so unless you mount a volume pre-populated with this file, it wilt take any effect until the container is restarted.

If you run the container detached from the console, you can watch the docker logs to see the status of the vagrant image as it starts up.

However you start the container, if there is a Vagrantfile present in /vagrant, then the container will try to start the vagrant box that it defines.  It might take a few minutes (more if there is a large download required), so keep an eye on the docker logs for progress.


<b>More on ENVironment variables</b><br>
For convenience, I have included Environment variables in the docker image which  can be used to provide quick control over the CPU, RAM and Disk assigned to the vagrant box that runs inside the docker container.  The environment variables are listed below in the table, you can either these manually from the command line when creating the container (see examples provided above and below), or in Portainer where they will show up automatically in the container configuration page when re-deploying the image.  (<b>Note:</b> Unless you make special provisions for persistence of data inside the docker container, any changes made to ENVironment variables will result in loss of work/data as re-applying ENVironment variables required deploying a replacement container.)
<br>

The ENVironment variables are as follows:

<table><tbody>

<tr><td style="text-align:center"; colspan="1" col width=50%><b>ENV Variable</b></td>
<td style="text-align: center;" colspan="1" col width=50%><b>values </b>(default in <b>bold</b>)</td></tr>

<tr><td style="text-align: center;">CPU</td>
<td style="text-align: center;">number of cpu cores e.g. <b>4</b></td></tr>

<tr><td style="text-align: center;">MEMORY</td>
<td style="text-align: center;">number of megabytes e.g. <b>4096</b></td></tr>

<tr><td style="text-align: center;">DISK_SIZE</td>
<td style="text-align: center;">number of Gigabytes e.g. <b>50</b></td></tr>

</tbody></table>

<b>Starting a pre-populated container</b><br>
The process is similar to the above, except you change the tag on the image name as per the example below.  To start a Windows 10 vagrant box with 4 CPU, 4GB RAM and 50 GB disk would look like this on the command line :

````
docker run --privileged -it --cgroupns host --name windows10 --device=/dev/kvm --device=/dev/net/tun -v /sys/fs/cgroup:/sys/fs/cgroup:rw -p 0.0.0.0:3389:3389/tcp  -e MEMORY=4096 -e CPU=4 -e DISK_SIZE=50 --cap-add=NET_ADMIN --cap-add=SYS_ADMIN gregewing/windows_on_linux:peru_windows-10-enterprise-x64-eval bash
````
Be sure to select the image you want by altering the tag on the docker image name.

Once the container is pulled and started, there will be a short delay before you will be able to connect to the image using RDP. In my experience this is around 5 minutes the first time a container is deployed, but restarted containers are up in around 30 seconds.  This is because of the work Vagrant has to do to set up the runtime environment. You can monitor Vagrants progress in the docker logs.  Once Vagrant has finished, your windows box is up and running and you should be able to connect to it using your choice of RDP client. 

Enjoy.

<b>Special Notes and Problem Solving:</b><br>
I found that I needed to explicitly set the group namespace to get around a problem I was having with docker (support ticket <a href="https://github.com/docker/buildx/issues/2183#top">here</a>).  Explicitly setting the cgroup namespace with <code> --cgroupns host</code> fixes that. 

If you are still having problems getting images to start, and you are sure they should work, then it might be the apparmor policies on the host. I recommend looking there for the culprit first.  My unscientific approach was to turn off apparmor while testing, you might want to be a bit more fine-grained in your approach.

Lastly, one of my test machines is an old Lenovo Q180, with an Intel Atom D2700 processor in it, which does not support VT-x, so Vagrant is completely unable to make use of it through docker, or even if it were installed locally.  It's a great little machine, and it runs docker containers beautifully, but not containers with embedded hypervisors. The point being, check that your hardware supports Virtualisation Extensions.

<b>Acknowledgements</b><br>
I did not do all this myself, I took inspiration from the following articles and other wonderfully clever people:

https://medium.com/axon-technologies/installing-a-windows-virtual-machine-in-a-linux-docker-container-c78e4c3f9ba1

https://github.com/vaggeliskls/windows-in-docker-container

https://github.com/SecurityWeekly/vulhub-lab
