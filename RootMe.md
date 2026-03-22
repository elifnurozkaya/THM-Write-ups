# Root Me - Writeup

### **Level:** Easy

## **Task 1: Deploy the machine**
The machine was started after connecting via **OpenVPN**.

---

## **Task 2: Reconnaissance**
The goal for this task is to gather machine information using tools like `nmap` and `gobuster`.

### **Nmap Scan**
An initial nmap scan was performed:
`nmap –sC –sV <ip>`

![Nmap Scan Results](images/nmap_scan.png)

* **-sC**: Scans using default nmap scripts.
* **-sV**: Pulls version information for open ports.

**Scan Results:**
* **Open Ports:** 2 ports are open: **22** (running **ssh**) and **80**.
* **Apache Version:** 2.4.41.

### **Directory Discovery**
**GoBuster** was used to find hidden directories on the web server:
`gobuster dir -u http://<ip> -w /usr/share/wordlists/dirb/common.txt`

![GoBuster Results](images/gobuster_results.png)

**Findings:**
Two subdomains appeared useful: `/panel` and `/uploads`. The hidden directory is **/panel**.

---

## **Task 3: Getting a Shell**
The `/panel` page allows for file uploads, which can be exploited using a **PHP reverse shell**.

1.  **Preparation:** The shell script was downloaded from [PentestMonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell).
2.  **Configuration:** The IP was updated to the local machine's IP and the port was set to **9999**.
3.  **Bypass:** Since `.php` files were not accepted, the extension was changed to **.php5** to allow the upload.
![Shell Upload Bypass](images/upload_bypass.png)

4.  **Listener:** A listener was started using **netcat**:
    `nc –nlvp 9999`
5.  **Execution:** The script was executed by navigating to the `/uploads` directory and clicking the uploaded file.

**User Flag:**
After gaining a shell, the flag was found at `/var/www/user.txt`.
![User Flag](images/user_flag.png)

---

## **Task 4: Privilege Escalation**
To elevate privileges to **root**, a search for files with **SUID** permissions was conducted:

`find / -type f –user root –perm –4000 2> /dev/null`

![SUID Search Results](images/suid_results.png)

**Exploitation:**
**Python** was identified in the SUID list. Using [GTFOBins](https://gtfobins.org/gtfobins/python/#shell) as a reference, the following command was used to escalate privileges:

`python -c 'import os; os.execl("/bin/sh", "sh", "-p")'`

**Root Flag:**
After gaining root access, the final flag was located at `/root/root.txt`.
![Root Flag](images/root_flag.png)
