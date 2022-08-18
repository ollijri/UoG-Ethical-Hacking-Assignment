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

</details>

