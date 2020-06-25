# ssh-into-google-colab
### Ever wondered how to leverage the computing power of  google colab in your local shell?
The following code does exactly that!
And the best part... you get root access to the entire VM!

Follow this link to know more: https://stackoverflow.com/questions/48459804/how-can-i-ssh-to-google-colaboratory-vm

* First log in and create a new notebook in [Google Colab](https://colab.research.google.com)
![Create New notebook](https://i.imgur.com/Sx7gUDj.png)
* Copy and paste the following code in the notebook.

### This will install **openssh-server** for the backend.
```
!sudo apt install openssh-server
```
### Now go to Google Colab notebook. Copy and paste the following code and run. This will connect colab backend to ngrok tunnel.
```
import random, string, urllib.request, json, getpass

#Generate root password
password = ''.join(random.choice(string.ascii_letters + string.digits) for i in range(20))
#Or use your custom password ;) easier ikr!
# password = 'plaintext'

#Download ngrok
! wget -q -c -nc https://bin.equinox.io/c/4VmDzA7iaHb/ngrok-stable-linux-amd64.zip
! unzip -qq -n ngrok-stable-linux-amd64.zip

#Setup sshd
! apt-get install -qq -o=Dpkg::Use-Pty=0 openssh-server pwgen > /dev/null

#Set root password
! echo root:$password | chpasswd
! mkdir -p /var/run/sshd
! echo "PermitRootLogin yes" >> /etc/ssh/sshd_config
! echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config
! echo "LD_LIBRARY_PATH=/usr/lib64-nvidia" >> /root/.bashrc
! echo "export LD_LIBRARY_PATH" >> /root/.bashrc

#Run sshd
get_ipython().system_raw('/usr/sbin/sshd -D &')

#Ask token
print("Copy authtoken from https://dashboard.ngrok.com/auth")
authtoken = getpass.getpass()
#or you can directly assing authtoken here
# authtoken = "..."

#Create tunnel
get_ipython().system_raw('./ngrok authtoken $authtoken && ./ngrok tcp 22 &')

#Get public address and print connect command
with urllib.request.urlopen('http://localhost:4040/api/tunnels') as response:
  data = json.loads(response.read().decode())
  (host, port) = data['tunnels'][0]['public_url'][6:].split(':')
  print(f'SSH command: ssh -p{port} root@{host}')

#Print root password
print(f'Root password: {password}')
```
    * This will ask for an Authtoken. Well ok, now where do i get that? Read on..
   ![Imgur](https://i.imgur.com/TlcbPdI.png)
    
### Goto [ngrok dashboard](https://dashboard.ngrok.com) and setup an account(if not already have).
   * What is ngrok you ask?
        * in simpl terms...
   ![Imgur](https://i.imgur.com/TN4ckcS.png)
        * in complex words...
   ![ngrok](https://i.imgur.com/5IfrRRr.png)
    * Click "Your Authtoken" on left pane.
   ![Imgur](https://i.imgur.com/TMwJllg.png)
    * Copy the Authtoken that appears(you're gonna need that!).

### Last few steps!!
    * Paste the copied Authtoken. (If for some reasons it fails, run it again)
### Ta da! And we're done here! You should be greeted with this message now.
   ![Imgur](https://i.imgur.com/KVnKh3F.png)

### Now just head over to your terminal and use those credentials to ssh into colab!!
   ![Imgur](https://i.imgur.com/7tzdlqA.png)

   ![Imgur](https://i.imgur.com/syQrCmy.png)

### Bonus! You can also attach your google drive(and Mega drives too!) with Google colab!!
