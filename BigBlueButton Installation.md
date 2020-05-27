1. Update your server Anchor link for: 1 update your server
First, make sure your server is up-to-date with latest packages and security updates.
Login to your server via SSH. You need to have an account that can execute commands as root (via sudo). Once logged in, first ensure that you have xenail multiverse in your /etc/apt/sources.list by doing the following
$ grep "multiverse" /etc/apt/sources.list
After entering the above command you should see an uncommented line for the multiverse repository, which may look like either this
deb http://archive.ubuntu.com/ubuntu xenial multiverse
or this
deb http://archive.ubuntu.com/ubuntu xenial main restricted universe multiverse
Don’t worry if your hostname in the URL is different from the above, what’s important is you see an uncommented link that contains multiverse. If you don’t, run the following command to add the multiverse repository to your /etc/apt/sources.list file.
$ echo "deb http://archive.ubuntu.com/ubuntu/ xenial multiverse" | sudo tee -a /etc/apt/sources.list
If you are a developer installing BigBlueButton on a VM for testing and development, some of BigBlueButton’s components, such as Tomcat, need a source of entropy when starting up. In a VM the available entropy can run low Tomcat can block for a long periods of time (sometimes minutes) before finishing its start-up. To give the VM lots of entropy, install a packaged called haveged (a simple entropy daemon):
$ sudo apt-get install haveged
If you are curious on the details behind entropy, see this link.
There are two applications needed by BigBlueButton: ffmpeg (create recordings) and yq (update YAML files). The default version of ffmpeg in Ubuntu 16.04 is old and yq does not exist in the default repositories. Therefore, before you install BigBlueButton, you need to add the following personal package archives (PPA) to your server to ensure you get the proper versions installed.
$ sudo add-apt-repository ppa:bigbluebutton/support -y
$ sudo add-apt-repository ppa:rmescandon/yq -y
Next, upgrade your server to the latest packages (and security fixes).
$ sudo apt-get update
$ sudo apt-get dist-upgrade
If you haven’t updated in a while, apt-get may recommend you reboot your server after dist-upgrade finishes. Do the reboot now before proceeding to the next step.
BigBlueButton HTML5 client uses MongoDB, a very efficient database used to synchronize state of the clients. To install MongoDB, do the following
$ wget -qO - https://www.mongodb.org/static/pgp/server-3.4.asc | sudo apt-key add -
$ echo "deb [ arch=amd64,arm64 ] http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.4.list
$ sudo apt-get update
$ sudo apt-get install -y mongodb-org curl
The BigBlueButton HTML5 client requires a nodejs server. To install nodejs, do the following
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
2. Install apt-get key for BigBlueButton repositoryAnchor link for: 2 install apt get key for bigbluebutton repository
All packages for BigBlueButton are digitally signed with the project’s public key. Before installing BigBlueButton, you need to add the project’s public key to your server’s key chain. To do this, enter the following command:
$ wget https://ubuntu.bigbluebutton.org/repo/bigbluebutton.asc -O- | sudo apt-key add -
If you are updating your server from BigBlueButton 2.0 (or earlier version), you need to first remove the bbb-client package.
$ sudo bbb-conf --stop
$ sudo apt-get purge -y bbb-client
This is because some files owned by bbb-client have moved to be owned by bbb-web. Deleting the bbb-client package before the upgrade to BigBlueButton 2.2 will allow bbb-web to create these files without conflict from the older version of bbb-client.
Next, your server needs to know where to download the BigBlueButton 2.2 packages. To configure the package repository, enter the following command:
$ echo "deb https://ubuntu.bigbluebutton.org/xenial-22/ bigbluebutton-xenial main" | sudo tee /etc/apt/sources.list.d/bigbluebutton.list
Next, run apt-get to pull down the links to the latest BigBlueButton packages.
$ sudo apt-get update
3. Back up custom configurationsAnchor link for: 3 back up custom configurations
If this is a new install you can skip this step.
If you are upgrading from BigBlueButon2.0, or an earlier release of BigBlueButton 2.2, and have made any custom changes, such as
•	set up your own SSL certificate in /etc/nginx/sites-available/bigbluebutton,
•	configured FreeSWITCH to accept incoming phone calls,
•	changed the default /var/www/bigbluebutton-default/default.pdf file
or any other changes outside of using bbb-conf, then you’ll want to backup these changes now before upgrading BigBlueButton. After you upgrade BigBlueButton, you can re-apply the custom configurations to your server.
4. Install BigBlueButtonAnchor link for: 4 install bigbluebutton
Note: If you are updating from BigBlueButton 2.0 (or earlier), do sudo apt-get purge bbb-client to uninstall bbb-client before installing this newer version.
We’re now ready to install BigBlueButton. Enter the following two commands
$ sudo apt-get install bigbluebutton
$ sudo apt-get install bbb-html5
For each command, when prompted to proceed, type ‘Y’ and press ENTER.
Note 1: You can ignore any errors “Failure to download extra data files” for the ttf-mscorefonts-installer package. This is a known issue with Ubuntu 16.04.
Note 2: If the installation exits with an error before finishing, doulbe-check the steps in Before you install. If you find and resolve any configuration errors, you can attempt to finish the installation using the command sudo apt-get install -f.
Note 3: If you still get errors after sudo apt-get install -f, stop here. The install has not finished and BigBlueButton will not run. See the troubleshooting guide and other options for getting help.
After the installation finishes, you can make the HTML5 the default client (recommended unless you need the Flash client).
Finally, to ensure all the packages are up-to-date, do one final dist-upgrade
$ sudo apt-get dist-upgrade
After the installation finishes, you can make the HTML5 the default client (recommended).
Next, restart BigBlueButton:
$ sudo bbb-conf --restart
This will restart all the components of the BigBlueButton server in the proper order. Note: Don’t worry if you initially see # Not running: tomcat7 or grails or Error: Could not connect to the configured hostname/IP address as the startup takes a few moments.
After the restart finishes, check the setup using bbb-conf --check. When you run this command, you should see output similar to the following:
$ sudo bbb-conf --check

BigBlueButton Server 2.2.0 (1571)
                    Kernel version: 4.4.0-142-generic
                      Distribution: Ubuntu 16.04.6 LTS (64-bit)
                            Memory: 16432 MB

/usr/share/bbb-web/WEB-INF/classes/bigbluebutton.properties (bbb-web)
       bigbluebutton.web.serverURL: http://178.128.233.105
                defaultGuestPolicy: ALWAYS_ACCEPT

/etc/nginx/sites-available/bigbluebutton (nginx)
                       server name: 178.128.233.105
                              port: 80, [::]:80
                    bbb-client dir: /var/www/bigbluebutton

/var/www/bigbluebutton/client/conf/config.xml (bbb-client)
                Port test (tunnel): rtmp://178.128.233.105
                              red5: 178.128.233.105
              useWebrtcIfAvailable: true

/opt/freeswitch/etc/freeswitch/vars.xml (FreeSWITCH)
                       local_ip_v4: 178.128.233.105
                   external_rtp_ip: stun:stun.freeswitch.org
                   external_sip_ip: stun:stun.freeswitch.org

/opt/freeswitch/etc/freeswitch/sip_profiles/external.xml (FreeSWITCH)
                        ext-rtp-ip: $${local_ip_v4}
                        ext-sip-ip: $${local_ip_v4}
                        ws-binding: :5066
                       wss-binding: :7443

/usr/local/bigbluebutton/core/scripts/bigbluebutton.yml (record and playback)
                     playback_host: 178.128.233.105
                 playback_protocol: http
                            ffmpeg: 4.1.1-0york1~16.04

/etc/bigbluebutton/nginx/sip.nginx (sip.nginx)
                        proxy_pass: http://178.128.233.105:5066


** Potential problems described below **
Any output that followed Potential problems may indicate configuration errors or installation errors. In many cases, the messages will give you recommendations on how to resolve the issue.
You can also use sudo bbb-conf --status to check that all the BigBlueButto processes have started and are running.
$ sudo bbb-conf --status
red5 ——————————————————► [✔ - active]
nginx —————————————————► [✔ - active]
freeswitch ————————————► [✔ - active]
redis-server ——————————► [✔ - active]
bbb-apps-akka —————————► [✔ - active]
bbb-transcode-akka ————► [✔ - active]
bbb-fsesl-akka ————————► [✔ - active]
tomcat7 ———————————————► [✔ - active]
mongod ————————————————► [✔ - active]
bbb-html5 —————————————► [✔ - active]
bbb-webrtc-sfu ————————► [✔ - active]
kurento-media-server ——► [✔ - active]
etherpad ——————————————► [✔ - active]
bbb-web ———————————————► [✔ - active]
bbb-lti ———————————————► [✔ - active]
At this point, your BigBlueButton server is listening to an IPV4 address. For example, if your server is at IP address 178.128.233.105, you can open http://178.128.233.105/ and see the welcome screen.
 
However, you can’t login from this screen unless you install the API demos (you’ll get a 404 error if you try it – the next step shows how to add the API demos).
If you intend to use this server with another front-end, you don’t need the API demos. You can integrate BigBlueButton with one of the 3rd party integrations by providing the integration the server’s address and shared secret. You can use bbb-conf to dispaly this information using bbb-conf --secret.
$ sudo bbb-conf --secret

       URL: http://178.128.233.105/bigbluebutton/
    Secret: 330a8b08c3b4c61533e1d0c5ce1ac88f

      Link to the API-Mate:
      http://mconf.github.io/api-mate/#server=http://178.128.233.105/bigbluebutton/&sharedSecret=330a8b08c3b4c61533e1d0c5ce1ac88f
5. Install API demos (optional)Anchor link for: 5 install api demos optional
The API demos are a set of Java Server Pages (JSP) that implement a web-based interface to test the BigBlueButton API.
To install the API examples, enter the following command:
$ sudo apt-get install bbb-demo
Once installed, you’ll be able to enter your name on the home page and click ‘Join’.
 
This will join you into the default meeting called “Demo Meeting”. Here’s a screen shot joining using FireFox, opening the Shared Notes panel, drawing on the whiteboard, and sharing a webcam.
 
When you are done with the API examples, you can remove them with
$ sudo apt-get remove bbb-demo
6. Restart your serverAnchor link for: 6 restart your server
You can restart and check your BigBlueButton server at any time using the commands
$ sudo bbb-conf --restart
$ sudo bbb-conf --check
The bbb-conf --check scans some of the log files for error messages. Again, any output that followed Potential problems may indicate configuration errors or installation errors. In many cases, the messages will give you recommendations on how to resolve the issue.
Notice that sudo bbb-conf --check warns you the API demos are installed, which enable anyone with access the server to launch a session (see removing API demos).
If you see other warning messages check out the troubleshooting installation.

