![Untitled](https://github.com/user-attachments/assets/7439ed00-f085-47fe-bc62-303fda5a855b)
----
# Understanding False Positives in SIEM: A Real-Life Walkthrough

## **Introduction**  
Not Every Alarm Means Danger 🚨🔥  

In the world of cybersecurity, alerts are like smoke detectors. They go off when something might be wrong. But sometimes? It’s just burnt toast.  

In this article, I'm walking you through a real investigation I did using the Let’s Defend SIEM tool to demonstrate how a harmless situation triggered a security alert. It’s a sweet and simple lesson about false positives that every budding SOC analyst should learn.  

### **What is a False Positive? 🤔**  
A false positive in cybersecurity is when a security system triggers an alert, thinking something is malicious, but it turns out to be harmless. Think of it like a guard dog barking at the mailman—an overreaction, but no real threat.  

**Common Causes of False Positives:**  
- Poorly written detection rules  
- Benign apps behaving like malware  
- Legacy systems generating unusual traffic  
- Misconfigured tools  
- Routine system maintenance tasks  

In a SOC (Security Operations Center), identifying false positives quickly is key to staying focused on real threats 🛡️.  

### **The Alert: Breaking It Down 🧩**  

![1746380843606](https://github.com/user-attachments/assets/044d023d-b3a3-4a68-b521-1414af9b4263)

Here are the alert details from the SIEM tool:  
- **EventID:** 117  
- **Event Time:** Feb 27, 2022, 12:36 AM  
- **Rule:** SOC167 - LS Command Detected in Requested URL  
- **Hostname:** EliotPRD  
- **Source IP:** 172.16.17.46  
- **Destination IP:** 188.114.96.15  
- **Requested URL:** https://letsdefend.io/blog/?s=skills  
- **User-Agent:** Firefox on Ubuntu Linux  
- **Device Action:** Allowed ✅  

So, why did this alert trigger? �  

That’s always the first question you should ask. Instead of rushing into packet captures or reputation checks, understand the logic behind the rule. In our case:  
- **Alert Trigger Reason:** URL Contains "LS"  

Now, what does "ls" mean?  

In Linux/Unix systems, `ls` is a command-line instruction used to list directory contents. In cyberattacks like Command Injection, an attacker may try to sneak commands like `ls`, `whoami`, or `cat /etc/passwd` into a web server to make it spill sensitive information.  

Some sample malicious URLs might look like:  
- `http://example.com/page.php?file=;ls`  
- `http://vulnsite.com/search?q=;ls -la`  

Looks scary, right? So let’s inspect the actual request...  

## **Step 1: Investigating the URL 🔎**  

![1746384541017](https://github.com/user-attachments/assets/0bb22d1d-f50b-4dd3-8168-7c27bbf80da9)

I navigated to the Log Management page in Let’s Defend and filtered HTTP traffic using the source IP `172.16.17.46`.  

The suspicious URL?  

![1746381106303](https://github.com/user-attachments/assets/e9bb9faf-e119-4d1c-9629-4184ad72184b)

`https://letsdefend.io/blog/?s=skills`  

Wait a minute... that "ls" isn’t a command. It’s just part of the word "skills." No shell commands here. It’s an innocent search query, likely typed by a human.  

## **Step 2: Is There Any Other Suspicious Activity? 🕵️♂️**  
Even though this alert looks harmless, part of being thorough is asking:  
- Is there any other suspicious traffic from the same source IP?  

I checked the logs for the source IP `172.16.17.46` and found nothing unusual.  
❌ No command injections  
❌ No directory traversal  
❌ No signs of exploitation attempts  

In fact, there was nothing suspicious about the URL requests at all. They were just normal, everyday requests to the Let’s Defend website—likely made by a regular user browsing or searching for something.  

![1746381256573](https://github.com/user-attachments/assets/b32ec1ee-e86c-47db-a109-58fe360b12b8)

![1746381326326](https://github.com/user-attachments/assets/c777e633-deb1-44f0-b348-005a64def572)

![1746381371321](https://github.com/user-attachments/assets/8fed8703-b099-4a66-8eba-478651ba5071)

![1746381400818](https://github.com/user-attachments/assets/af58b71d-d466-4c9d-a05c-8f4fbd3bb08c)


To confirm this, I also navigated to the Endpoint Security page and reviewed the logs for the host with **Hostname:** EliotPRD. Among other things I checked there, the browser history stood out: all requests were made solely to the `letsdefend.io` domain—no redirections, no calls to shady or external sites. Just clean, legit traffic.  

![1746385304899](https://github.com/user-attachments/assets/685ae312-f74d-4120-b88d-f88af9b0e151)

### **Step 3: Document Your Findings 📝**  
Every investigation must leave a paper trail. Here’s a summary snippet of the comment I left in the incident:  
> *"Triggered by URL pattern match. 'ls' appears in 'skills' – not a command. No other malicious behavior found from source IP. Alert confirmed as a false positive."*  

### **Step 4: Close the Alert as False Positive 🚪❌**  
Given the evidence:  
- The "ls" string in the URL was part of a legitimate word  
- No other malicious traffic was detected from the same IP  
- The device action was "allowed," and rightly so  

This alert was not a real attack. I closed it as a **Definite False Positive** 🛑.  

![1746382812073](https://github.com/user-attachments/assets/d94b1863-3984-4674-be66-b959854aeadd)

## **Conclusion: Why This Matters**  
As a SOC Analyst, you’ll be flooded with alerts 🚨. Not all of them are real threats. That’s why being able to:  
- Think critically  
- Investigate thoroughly  
- And document clearly  

...is just as important as knowing how to write scripts or scan logs.  

False positives can occur due to:  
- Basic keyword triggers (like "ls" in "skills")  
- Legacy detection rules  
- Misconfigured SIEM settings  

This investigation taught (and reminded) me that cybersecurity isn’t just about fighting hackers—it’s about telling the difference between barking dogs and real burglars.  
