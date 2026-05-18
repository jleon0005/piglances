<h1 align="center">GLANCES IMPLEMENTATION ON RASPBERRY PI</h1>
<hr>
# INTRODUCTION
I made this small implementation so I could monitor hardware resources on a website rather than accessing htop on terminal.
This was done on a Raspberry PI 4, but I guess it could be done on any model
# STEP BY STEP GUIDE

## Step 1 - Update and glances installation.
+ Execute update
`sudo apt update`
+ Install Glances
`sudo apt install glances -y`

> [!IMPORTANT]
> For what I've experienced, running "sudo apt install glances -y" normally does not perform a good installation, due to broken packages errors or incompatibilities, for this type of issues refer to steps 1.2 - Alternative download methods[^1].

## Step 2 Fire up glances
`glances -w`
What happens: It starts a web server on port 61208.
Test it: Open your browser on any device on your network and go to http://192.168.1.2:61208. You should see the dashboard.
Stop it: Press Ctrl+C in the terminal to stop it for now. We'll make it run automatically later.

## Step 3 Make Glances Start Automatically
Right now, if you reboot the Pi, Glances stops. Let's make it a service so it runs in the background forever.
+ Create the Service File
`sudo nano /etc/systemd/system/glances-web.service`
+ Paste this in:
------------------------------------------------------------
[Unit]
Description=Glances Web Server
After=network.target

[Service]
Type=simple
User=pi  # Change 'pi' to your actual username if different
ExecStart=/usr/bin/glances -w
Restart=always
RestartSec=10

[Install]
WantedBy=multi-user.target
------------------------------------------------------------

> [!CAUTION]
> Change User=pi to whatever username you use on the Pi. If you're logged in as admin, put admin.

+ Enable and Start the Service
`sudo systemctl daemon-reload`
`sudo systemctl enable glances-web`
`sudo systemctl start glances-web`

+ Check if it's running:
`sudo systemctl status glances-web`

## Step 4 - Final verification
Open your browser.
Navigate to http://192.168.1.x:61208.
You should see the Glances dashboard.

[^1]: Alternative methods and errors.## Troubleshooting
+ 502 Bad Gateway: Apache can't reach Glances.

Check if Glances is running: sudo systemctl status glances-web.
Check if port 61208 is listening: sudo netstat -tulpn | grep 61208.
Ensure the ProxyPass line in Apache matches the port exactly.

+ Permission Denied: Glances might be trying to read system stats it can't access.

Try running the service as root (not recommended for security, but good for testing): Change User=pi to User=root in the service file.
Better fix: Add your user to the adm group: sudo usermod -aG adm your_username.

+ Slow Loading:
The Pi 4 is fast, but if you have tons of processes, Glances might lag. It's usually fine though.

## Step 1.2 - Alternative installation method
> [!NOTE]
> ONLY use this method in case you get the following error message or similar
> `File "/usr/lib/python3/dist-packages/starlette/staticfiles.py", line 56, in __init__
raise RuntimeError(f"Directory '{directory}' does not exist")
RuntimeError: Directory '/usr/lib/python3/dist-packages/glances/outputs/static/public' does not exist`

+ In case you've already attempted to install glances and got an issue on running `glances -w` you have the following 2 alternatives for installing Glances to run after cleaning up the previous installation.
  
### Clean up the broken install
`sudo apt remove glances -y`

`sudo apt autoremove -y`
### Install Python Dependencies
Python dependencies - `sudo apt install python3-pip python3-venv python3-dev build-essential -y`

> [!WARNING]
> Installing via PIP may trigger a Raspberry PI guardrail indicating not to mess up with system files via pip (Understandable)
> However, for this case since this implementation is for a local PI for monitoring the risk is low.
> We have 2 alternatives, installing it the clean way, or the direct (more risky) way.

### Alternative 1 - Clean way way (Virtual environment)

+ Create a virtual environment
`sudo mkdir -p /opt/glances`
`sudo chown $USER:$USER /opt/glances`
`cd /opt/glances`
`python3 -m venv venv`

+ Activate and install
`source venv/bin/activate`
`pip install glances`

+ Update the Systemd Service
You need to point the service to the Python inside the venv, not the global one. Edit the service file:
`sudo nano /etc/systemd/system/glances-web.service`

+ Change the ExecStart line to:
`ExecStart=/opt/glances/venv/bin/glances -w`

+ Make sure User=pi is still correct
Save, reload, and start:
`sudo systemctl daemon-reload`
`sudo systemctl restart glances-web`

### Alternative 2 - Force it way
Best for local tools where you control the box. Just adds the flag to bypass the check.

+ Install with the override flag Run this command:
`sudo pip3 install --break-system-packages glances`

+ Verify Installation Check that it installed correctly:
`glances --version`

+ Test web server
`glances -w`

