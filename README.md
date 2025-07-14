# **Orange-Pi-Honey-Pot**
Docker-Based Multi-Honeypot Stack, Built with Linux

1. Create a dedicated user for Cowrie (using sudopy)
sudo adduser --disabled-password --gecos "" sudopy

2. Install dependencies
sudo apt install -y git python3-venv python3-pip libssl-dev libffi-dev build-essential authbind


3. Clone Cowrie repo (as sudopy)
sudo -u sudopy -H bash -c "cd ~ && git clone https://github.com/cowrie/cowrie.git"
Note: This will now clone Cowrie into /home/sudopy/cowrie.

4. Set up virtual environment (as sudopy)
sudo -u sudopy -H bash -c "cd ~/cowrie && python3 -m venv cowrie-env && source cowrie-env/bin/activate && pip install --upgrade pip && pip install -r requirements.txt"
This step (especially pip install -r requirements.txt) will take the longest. Let it run completely.

5. Confirm Cowrie Repository Location: First, let's quickly verify that the cowrie repository was indeed cloned into /home/sudopy/cowrie as expected, given the previous issues.
ls -l /home/sudopy/cowrie/src/cowrie/start.py

**PROBLEM**

6. ls -l /home/sudopy/cowrie/src/cowrie/start.py
ls: cannot access '/home/sudopy/cowrie/src/cowrie/start.py': Permission denied

**SOLUTON**

7. ls -ld /home/sudopy
drwx------ 4 sudopy sudopy 4096 Jul 13 13:48 /home/sudopy
Change permissions of /home/sudopy: sudo chmod 755 /home/sudopy

8. And finally, re-check the start.py path:
ls -l /home/sudopy/cowrie/src/cowrie/start.py

**IF DIDN’T WORK**

9. Confirm sudopy's exact home directory (again, just to be 100% sure after the usermod command):
grep sudopy /etc/passwd

10. This will output a line for sudopy showing its username, UID, GID, GECOS, home directory, and shell. Confirm the home directory is /home/sudopy.
Ensure /home/sudopy has the correct permissions and ownership (even though we did this, it's good to re-confirm before proceeding):
sudo chown sudopy:sudopy /home/sudopy
sudo chmod 755 /home/sudopy

11. Remove the existing cowrie directory as sudopy: This command will remove the potentially incomplete or problematic cowrie directory.
sudo -u sudopy rm -rf /home/sudopy/cowrie

12. Perform a clean clone of the Cowrie repository as sudopy: This will ensure all files are correctly downloaded into the designated location with proper ownership.
sudo -u sudopy git clone https://github.com/cowrie/cowrie.git /home/sudopy/cowrie

13. Now, run the virtual environment setup and dependency installation again:
sudo -u sudopy -H bash -c "cd /home/sudopy/cowrie && python3 -m venv cowrie-env && source cowrie-env/bin/activate && pip install --upgrade pip && pip install -r requirements.txt"

**PROBLEM**
14. sudo -u sudopy git clone https://github.com/cowrie/cowrie.git /home/sudopy/cowrie
Cloning into '/home/sudopy/cowrie'...
fatal: Unable to read current working directory: No such file or directory

15. Now, become the sudopy user and clone:
sudo su - sudopy

16. You should now be logged in as sudopy. Your prompt might change to sudopy@yourhostname:~$
Once you are the sudopy user, from the sudopy shell, run the clone command:
git clone https://github.com/cowrie/cowrie.git cowrie

17. This will clone cowrie into ~/cowrie, which for sudopy is /home/sudopy/cowrie. This is safer as git is running directly within sudopy's home directory.
Watch for the output of this git clone command. It should show the cloning progress.
Exit the sudopy shell: Once the git clone completes successfully, type exit to return to your original user account.
exit

18. Ensure /home/sudopy has correct permissions and ownership: (You confirmed these were successful and silent, which is good. Let's just state them for completeness).
sudo chown sudopy:sudopy /home/sudopy
sudo chmod 755 /home/sudopy

19. Remove any lingering (and problematic) /home/sudopy/cowrie directory:
sudo -u sudopy rm -rf /home/sudopy/cowrie

**SHOULD BE SUCCESSFUL**

**IF DIDNT**

20. sudo su - sudopy
Still from the sudopy user's prompt, set up the virtual environment and install dependencies:

cd cowrie && python3 -m venv cowrie-env && source cowrie-env/bin/activate && pip install --upgrade pip && pip install -r requirements.txt

21. Remove the existing cowrie directory:
rm -rf cowrie

22. Now, re-run the git clone command:

git clone https://github.com/cowrie/cowrie.git cowrie

23. After the git clone finishes successfully, run the virtual environment setup and install dependencies:
cd cowrie && python3 -m venv cowrie-env && source cowrie-env/bin/activate && pip install --upgrade pip && pip install -r requirements.txt

24. open cowrie.cfg for editing:
sudo nano /home/sudopy/cowrie/etc/cowrie.cfg

25. Add the line user = sudopy under the [honeypot] section (e.g., just below hostname = svr04), ensuring it's not commented out.
Changed the SSH listen_endpoints from tcp:2222:interface=0.0.0.0 to tcp:22:interface=0.0.0.0 under the [ssh] section.
Set the Telnet listen_endpoints under the [telnet] section to either tcp:23:interface=0.0.0.0 (if you set up authbind for port 23) or kept it as tcp:2223:interface=0.0.0.0.

26. Create the cowrie.service file
cat <<EOF | sudo tee /etc/systemd/system/cowrie.service

27. Then, immediately copy and paste the rest of the block:
[Unit]
Description=Cowrie SSH/Telnet Honeypot
After=network.target

[Service]
User=sudopy
Group=sudopy
ExecStart=/usr/bin/env /home/sudopy/cowrie/cowrie-env/bin/python /home/sudopy/cowrie/cowrie-env/bin/twistd -y /home/sudopy/cowrie/src/cowrie.tac --pidfile=/home/sudopy/cowrie/var/run/cowrie.pid
WorkingDirectory=/home/sudopy/cowrie
Restart=always
RestartSec=5s
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
EOF

28. Reload systemd to recognize the new service file:

sudo systemctl daemon-reload

29. Enable the Cowrie service to start automatically on boot:

sudo systemctl enable cowrie

30. Start the Cowrie service immediately:

sudo systemctl start cowrie

Now the honey pot is done

<img width="1470" height="956" alt="Screenshot 2025-07-13 at 11 29 07 PM" src="https://github.com/user-attachments/assets/a1903b2c-8656-4269-a185-3f3a7edec041" />

31. Create the directory (if it doesn't already exist): First, ensure you are in your home directory:

cd /home/sudopy/

32. Then, create the directory:

mkdir honeypots-docker
(If it already exists, this command will tell you "mkdir: cannot create directory 'honeypots-docker': File exists". That's fine, you can just proceed to the next step.)

33. Navigate into the new directory:

cd honeypots-docker

34. Create and edit the docker-compose.yml file using nano:

nano docker-compose.yml

Paste this 
version: '3'
services:
  cowrie-ssh:
    image: cowrie/cowrie
    container_name: cowrie
    ports:
      - "22:2222"
    environment:
      - COWRIE_USER=cowrie
    restart: always

  dionaea:
    image: honeynet/dionaea
    container_name: dionaea
    ports:
      - "21:21"
      - "42:42"
      - "69:69/udp"
      - "135:135"
      - "443:443"
      - "445:445"
      - "1433:1433"
      - "3306:3306"
    restart: always

  honeytrap:
    image: mfontani/honeytrap
    container_name: honeytrap
    ports:
      - "80:80"
      - "8080:8080"
      - "3389:3389"
    restart: always

  elastic-logging:
    image: sebp/elk
    container_name: elk
    ports:
      - "5601:5601"
      - "9200:9200"
    restart: always

  **This ensure everything is running in the background as soon as you turn on the OrangePi.**


<img width="1470" height="956" alt="Screenshot 2025-07-13 at 11 40 25 PM" src="https://github.com/user-attachments/assets/c8ef6296-8a6c-4301-bce1-95f595afcf6d" />

**YOU WILL NOW SEE ANYTHING SOMEONE TRIES TO LOG INTO THROUGH SSH, HTTP**

<img width="1470" height="956" alt="Screenshot 2025-07-13 at 11 02 08 PM" src="https://github.com/user-attachments/assets/52d90f32-d3d0-4f81-ba8a-5fb8e91b1327" />

**The Code To Test your Honey Pot**
SQL:  mysql -h 100.116.43.53 -u root -P 3306
HTTP: http://100.116.43.53/index.html

**This Project taught Me**
- Threat Intelligence Gathering: Understanding Attacker Behavior, Real-time Intelligence, Identifying Motives and Targets
- Early Warning and Detection: Proactive Defense, Reduced False Positives
- Distraction and Diversion: Luring Attackers, Delaying Attacks
- Vulnerability Identification and Improvement: Revealing Weaknesses, Testing Security Measures
