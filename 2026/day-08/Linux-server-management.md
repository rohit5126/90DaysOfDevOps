# Today's goal is to deploy a real web server on the cloud and learn practical server management.

## create AWS cloud ec2 instance.

<img width="955" height="268" alt="image" src="https://github.com/user-attachments/assets/f5e57d97-4582-4ea9-b37c-136d2d800236" />

## connect via ssh using putty.

**download key file in .pkk format to use with putty. place the file in putty under Connection/ssh/auth/credentails. use ubuntu user to login**

<img width="485" height="287" alt="image" src="https://github.com/user-attachments/assets/f011493d-0d64-441b-a01d-38a35d795aa4" />

## update the server

**after first time connecting with the server, update the repo with local packages.**

**sudo apt update**

<img width="638" height="119" alt="image" src="https://github.com/user-attachments/assets/034fe091-323d-4646-9be2-f18cab550e04" />

## install nginx on the server

**sudo apt install nginx -y**

<img width="617" height="241" alt="image" src="https://github.com/user-attachments/assets/aeb71e1a-3f76-4226-ae4c-73ed7f6b0336" />

## install docker on the server

**sudo apt install docker.io -y** 

**after installing update user group to have docker to avoid any error while running docker commands**

**sudo usermod -aG docker $USER**

**sudo reboot**

/<img width="529" height="38" alt="image" src="https://github.com/user-attachments/assets/0e64a239-1677-474e-ab8e-a3fc4b7b27d6" />

## after installation upadte the inbound rules to listen at port 80 from anywhere.

<img width="950" height="367" alt="image" src="https://github.com/user-attachments/assets/22db4d02-533f-4822-a7d0-b2112783f1bd" />

## check on the server if port is listening

**nc -zv localhost 80**

**sudo netstat -tulnp | grep nginx** 

<img width="673" height="69" alt="image" src="https://github.com/user-attachments/assets/0db9627c-8e02-43f5-8fa2-8e47e3f71d81" />

## nginx webpage is accessible

**try to access nginx webpage using ip add of the server with port 80.  [ip address]:80**

<img width="650" height="470" alt="image" src="https://github.com/user-attachments/assets/2da3cdf9-53f0-48bc-8fc2-96e34e09bca3" />

## Extract and save logs in a file.

**use journal command to extract and save logs**

**journalctl -u nginx >> logs.txt**

<img width="677" height="52" alt="image" src="https://github.com/user-attachments/assets/7aa12edb-ca5e-4af7-a9fd-c66ec2cc9c8c" />

## download the file to local machine.

**we can use Secure copy protocol(SCP)**

**scp -i /path/to/your-key.pem ubuntu@ec2-ip-address:/remote/path/to/file /local/path/to/destination**

<img width="914" height="121" alt="image" src="https://github.com/user-attachments/assets/8d3374a0-f119-4f45-97aa-1843f11a227f" />

