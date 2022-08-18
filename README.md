## ToE Vulnerabilities and Exploitation

<details>
<summary> Flag 1 – Incorrect categorisation of website. </summary>
<p><p/>
To begin, using the output of the initial nmap scan as a base, it is known the ToE has an open port 80. Port 80 is used by HTTP and is the default network port used to send and receive unencrypted web pages. This alludes to the fact this ToE has a webserver running off this IP address.<br/>


HTTP does not use any form of encryption or certificates to secure it unlike port 443, ‘HTTPS’, so remains historically one of the most probed ports on the Internet and will form the basis of this ethical hack.


Within the website, accessed by putting the IP of the ToE into the URL bar, are messages posted by each user, all with categories set. It would appear if the category is not set it defaults to ‘uncategorised’ which if accessed can allow you to see messages you are potentially not authorised to see as seen below:

<p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185343935-6524d0de-03e4-4fc7-aa79-54b8840f9e14.png">
</p>

</details>

<details>
<summary> Flag 2 – Insecure Admin Page (CVE-2017-8917) </summary>
<p><p/>
Navigating to the directory ‘/administrator/’ found from a guess of common admin login directories, it is clear the site is running their content management system (CMS) using the web application ‘joomla!’.

<p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185345885-c1d4419b-2263-46d1-a73b-c3d02f66ac07.png">
</p>
  
 All joomla web pages after version 1.6.0 store their version number in an xml file accessible through the URL seen below:
  
  ```http://www.[thejoomlawebsite].com/administrator/manifests/files/joomla.xml```
  
  This is a major step in determining whether the page has any exploitable vulnerabilities as running out-of-date versions without the latest patches leaves the website open to attack.
  
  <p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185346752-72464ffc-ab94-4e31-b9c2-c2df1c334a40.png">
</p>

  On this same page, viewing the page source exposes the version number of joombla as “3.7.0”.
  
  Now that the version number is revealed it narrows down the vulnerabilities that are possible to exploit for this specific version of ‘joombla’. For this example, exploit CVE-2017-8917 was used, an SQL injection attack that allows attackers to execute arbitrary SQL commands via unspecified vectors.
  
Using the base of a command found on the exploit database website:

  ``` sqlmap -u "http://localhost/index.php?option=com_fields&view=fields&layout=modal&list[fullordering]=updatexml" --risk=3 --level=5 --random-agent --dbs -p list[fullordering] ```

  ‘sqlmap’ was loaded with the relevant ip for the ToE. When run, this spat out the database names of the DBMS that was being run revealing two databases 
  
From the two databases revealed from the previous command, ‘joombla’ was chosen to be targeted as it sounds like it would hold information of greater value than the alternative. Using this database as a target, the below command is used to dump all the information about the tables within the database as it can within the CLI. This includes a table known as “__users”, which as will be seen will be essential for grabbing user data.
  
  ![image](https://user-images.githubusercontent.com/66912443/185354231-da7dbec7-a5ee-494c-82e4-8811c6689c6a.png)
![image](https://user-images.githubusercontent.com/66912443/185354297-52d25268-f01e-4643-b508-32e2afc44c35.png)

 Running the program again, but specifically looking for that aforementioned table provides a more detailed view of what is going on inside. This leaves an output with 3 users, with now exposed emails. As the emails are not encrypted all that needs to be cracked is the passwords.
  
  The hashed passwords are put into a text document to be cracked by john. Using the beginning ID ‘$2y$’ it can be determined the passwords are encrypted using blowfish.
  
  <p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185355434-d25eb2a1-490a-47da-82ca-5ec0b1dfd68e.png">
</p>
  
  Now we have the hashes, a wordlist needs to be built to try crack them within reasonable time. To do this, the program ‘CeWl’ is used. The below command is used to set to crawl the website (via IP address) at a depth of 5 with a minimum character count of 3 and put each word found into a file called “passlist”.
  
  <p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185354999-ddf956fe-032d-4ecd-ac76-7bb005e22062.png">
</p>

  Now that a wordlist has been created, the program ‘john the ripper’ can be used to crack the hashes. Running the below command results in one hash getting successfully cracked and found to be the word “isaribi”. This belongs to the admin account ‘orga’, a super user .
  
   <p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185355739-f4954333-5cf6-40d4-a57a-39064d4417b2.png">
</p>

  To try and locate another password, john was run for a while without a password list, this cracked another hash revealing it to be the word “cookie”. This belongs to the admin account ‘Biscuit’.
  
  <p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185355946-c6a5beae-49b4-4571-b744-4080a334dc7e.png">
</p>
  
</details>

<details>
<summary> Flag 3 – Viewing other users files </summary>
  <p><p/>
  By leaving the home directory of ‘orga’ using “cd ..”, it is possible to see all other users that reside on this system. As seen below, under the acount ‘orga’ the user can change directory into another users home directory therefore allowing the viewing of all of their personal files. 
  
  <p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185356358-d4e77e28-d18e-42b8-b106-22a95c7539d8.png">
</p>

</details>

<details>
<summary> Flag 4 – Gaining SSH Access </summary>
<p><p/>
Using the username and password of the ‘super user’ account found in flag 2, it was possible to gain access to the ToE using remote ssh connection as the login for the CMS was the same as it was for ssh.

  
Username: orga@192.168.56.101  
Password: isaribi
  
  <p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185356872-4aea575b-5edd-43b9-8ef0-3af6ea979e50.png">
</p>
  
<p><p/>
Listing all directories gives the file “.Flag4”. Reading the contents gives the following:
  
  <p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185357491-4eeec548-70fa-46be-ab71-a15c4e40c0a5.png">
</p>

</details>

<details>
<summary>	Flag 5 – Root Access (CVE-2021-4034) </summary>
<p><p/>
The vulnerability chosen was ‘CVE-2021-4034’. This is a vulnerability of the ‘pkexec’ toolkit. This toolkit is dangerous enough as its purpose is to “allow unprivileged users to run commands as privileged users according to predefined policies”. This exploit makes use of this to cause an unauthorised local privilege escalation on the target machine.

Looking up the name in the ToE terminal confirms this toolkit is present.

<p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185359889-4333051e-6498-480f-8c5d-594106d1b9a8.png">
</p>

The exploit works by running the below script, created by Andris Raugulis on github. This script works as the version of pkexec on the ToE doesn't handle the calling parameters count correctly and ends trying to execute environment variables as commands. 

  <p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185360444-5032c5b2-32d4-4ff9-87df-1a16e49ba84d.png">
</p>
  
  On the ToE which is accessible through the ‘orga’ account, this is copied line for line and put in a file named ‘exploit.c’.
  
  Next step is to run the code with the following command, generating the executable ‘poc’:
  <p></p>
  
  ![image](https://user-images.githubusercontent.com/66912443/185360620-66e12e7b-f856-4a84-bc8a-16c2e5d07484.png)
  ![image](https://user-images.githubusercontent.com/66912443/185360640-cf027a9b-4746-4f7c-9fcc-626c85c3e428.png)

Once the ‘poc’ file is run, the user is provided with root access as seen below:
  <p></p>
  
  ![image](https://user-images.githubusercontent.com/66912443/185360880-8142754b-39b9-4331-ab3a-01a51191fbf3.png)

  This leads to being able to find Flag 5 in the ‘root’ directory.
  
  <p align="center">
  <img src="https://user-images.githubusercontent.com/66912443/185361007-6f9d0bb1-05f9-4696-a489-9c04f2f6f841.png">
</p>

</details>
