## Notes on the videos
<br />

<details>
<summary>Video: Install and Run Nexus on a Cloud Server</summary>
<br />

Login to your account on [DigitalOcean](https://cloud.digitalocean.com/login) and create a new Droplet having at least 4GB RAM (better 8GB). Create a firewall rule opening the ports 22 for SSH.

SSH into the server:
- copy the Droplet's IP address
- execute `ssh root@<droplet-ip-address>`

Install Java Version 8 (needed for Nexus):
- `apt update`
- `apt install openjdk-8-jre-headless`

Download and unpack the latest Nexus version into the /opt folder:
- `cd /opt`
- `wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz`
- `tar -zxvf latest-unix.tar.gz` => two folders nexus-3.46.0-01 and sonatype-work

Create a nexus user to be used to run the nexus application:
- `adduser nexus`

Change the privileges for the unpacked folders (nexus user needs to access both):
- `chown -R nexus:nexus nexus-3.46.0-01`
- `chown -R nexus:nexus sonatype-work`

Configure Nexus to run with the nexus user we just created:
- Add `run_as_user="nexus"` to the file `nexus-3.46.0-01/bin/nexus.rc` using vim

Switch to the nexus user and start Nexus:
- `su - nexus`
- `/opt/nexus-3.46.0-01/bin/nexus start`

Check the port on which Nexus is running:
- `ps aux | nexus` shows the PID
- `netstat -tlnp` shows that the process with the nexus PID is listening on port 8081

So go to the DigitalOcean admin webpage and add a firewall rule opening the port 8081 for all IP addresses.

Now you can access Nexus in your browser via `http://<droplet-ip-address>:8081`.

</details>

*****

<details>
<summary>Video: Introduction to Nexus</summary>
<br />

There is a predefined `admin` user. Its password is stored in `/opt/sonatype-work/nexus3/admin.password`.
You can log in with this password on the nexus website. You will be prompted to change the password. As soon as you've changed it, the file `/opt/sonatype-work/nexus3/admin.password` will be removed.

</details>

*****