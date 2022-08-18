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

</details>
