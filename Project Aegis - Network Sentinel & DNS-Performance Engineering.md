# **🧠 Adblocker Lab: Network Telemetry & Sinkhole Engineering

> **Objective:** **Engineering a DNS-Sinkhole with SOC Integration & Threat Intel**

---
## **📂 Project Roadmap & Progress** 

**Overview:**
In this project, I will deploy a network-wide DNS-Sinkhole (using **Pi-hole** or **AdGuard Home**) within a virtualized environment. The goal is to act as the "Network Gatekeeper," intercepting DNS queries to block ads, trackers, and known malicious domains. This project will conclude by piping "Blocked Request" telemetry into our existing Splunk SIEM to create a real-time "Threat Map" of the home network.

---

#### **Phase 1: Virtualized Infrastructure & Containerization**

_**Objective:** Deploy a high-availability DNS engine using Docker Desktop on Windows 11._
- [x] **WSL 2 Backend Hardening:** Resolved "Server Resolution" errors via manual kernel MSI installation.
    - Verified Virtualization (VT-x/SVM) is enabled in BIOS/UEFI.
- [x] **Infrastructure as Code (IaC):** Created `docker-compose.yml` in the `Aegis-Lab` directory.
    - Configured environment variables for the Pi-hole administrative password.
- [x] **Deployment:** Execute `docker-compose up -d` to pull the Pi-hole image and initialize the container.
    - Access the local dashboard via `http://localhost/admin`.

### **Phase 1: Virtualized Infrastructure**

_**Objective:** Deploy the sinkhole without dedicated hardware._
- [x] **The Environment:** Setting up a Linux-based Docker container or a small VirtualBox VM (1GB RAM) to host the DNS service.
- [x] **Networking:** Configuring a **Static IP** for the container/VM so it becomes the reliable "Phone Book" for the network.
- [x] **The Tool:** Installing **Pi-hole** via Docker or script.

### **Phase 2: DNS Traffic Orchestration**

_**Objective:** Directing network traffic through the Sinkhole._
- [x] **DHCP Configuration:** Accessing the home router to point the DNS settings to our new virtual machine.
- [x] **Fail-Safe Implementation:** Setting up a secondary, non-filtered DNS (like Cloudflare 1.1.1.1) to ensure the household doesn't lose internet if the lab is turned off.
- [x] **Verification:** Using `nslookup` commands to verify that ads are being "sinkholed" (sent to 0.0.0.0).

### **Phase 3: Threat Intelligence Integration**

_**Objective:** Moving beyond ad-blocking to actual Security._
- [x] **Blocklist Curation:** Integrating "OSINT" (Open Source Intelligence) feeds that track known Malware and Phishing domains.
- [x] **Regex Filtering:** Writing custom filtering rules to block specific telemetry pings from smart TVs or IoT devices.
- [x] **Validation:** Simulating a "Phishing Click" (using safe, test-only domains) to verify the sinkhole blocks the connection.

### **Phase 4: Splunk Telemetry Pipeline**

_**Objective:** Visibility and Monitoring._
- [x] **Syslog Forwarding:** Configuring the sinkhole to send its query logs to your **Windows Splunk Indexer**.
- [x] **Data Parsing:** Teaching Splunk how to read DNS logs (identifying which device is asking for which website).
- [x] **Dashboarding:** Creating a visual map in Splunk showing "Top Blocked Domains" and "Most Active Devices."

### **Phase 5: Incident Response & Automation**

_**Objective:** Active Defense._
- [x] **The Alert:** Setting up a Splunk alert for when a device tries to reach a "High Risk" domain.
- [x] **The Python Auditor:** Enhancing your previous Python script to automatically check the health of the DNS-Sinkhole and restart the container if it stops responding.

---
### **📝 Resource Library**
- **The Only Docker Tutorial You Need To Get Started:** [Watch Here](https://www.youtube.com/watch?v=DQdB7wFEygo&t=175s)
- **Pi-hole Made EASY - A Complete Tutorial:** [Watch Here](https://www.youtube.com/watch?v=e_EfmKdP2ng)

---
# **Phase 1: Deploying the Containerized DNS Resolver**

## **1.1 Setting the Stage**

The goal for this phase was to get a stable, isolated DNS engine running without cluttering my main OS. I went with **Docker Desktop** using the **WSL 2 (Windows Subsystem for Linux)** backend because it’s the most efficient way to run Linux-based security tools on Windows. The idea was to build this using **Infrastructure as Code**, so I could spin it up or down whenever I needed it without having to re-configure everything manually.

## **1.2 Dealing with the WSL 2 "Wall"**

Right out of the gate, I hit a snag. When I tried to update the WSL kernel through the command line, it threw a `Server name or address could not be resolved` error. This was a bit ironic—a network error while trying to build a networking tool.

- **My Fix:** Instead of wasting hours fighting the CLI, I just manually grabbed the **WSL 2 Linux Kernel Update MSI** from Microsoft’s site.
- **The Result:** It worked immediately. Sometimes the manual route is just faster than trying to "fix" an automated tool that’s acting up.

## **1.3 The Password Loop (Troubleshooting Persistence)**

After I got the **DNS Resolver** running, I ran into a weird bug where the web dashboard wouldn't let me in. I’d use the command line to set a new password, it would say "Success," but then the website would tell me I was wrong.

I realized this was likely a **Volume Persistence** issue. Basically, Docker was holding onto some old, messy configuration files in my local folders, and no matter what I did in the terminal, the container kept "remembering" the old state.

- **The "Nuclear" Solution:** I shut down the container, manually deleted the local folders where it stores its data (`etc-pihole`), and simplified my `docker-compose.yml` file to get rid of any hard-coded password variables.
- **The Outcome:** I restarted the container, set the password one last time via the terminal, and I was finally in. This taught me a lot about how Docker "remembers" things even when you think you've cleared the slate.

---
# **Phase 2: Routing the Traffic & Fixing VPN Issues**

## **2.1 Setting the "Phonebook"**

With the Resolver finally working, it was time to actually use it. I had to tell my Windows laptop to stop using the default DNS and instead use the container I just built. I went into my **IPv4 settings** and set the DNS to `127.0.0.1`. This is the "loopback" address, which basically tells my laptop: _"If you're looking for a website, ask the container running right here."_ * **Safety Step:** I kept `9.9.9.9` as my secondary DNS. I didn't want my whole internet to break if I decided to turn off Docker for a bit.

## **2.2 VPN Interference**

I usually run a VPN, and I noticed it was causing some friction. VPNs like to hijack all your traffic into an encrypted "tunnel," which can sometimes hide your own local services (like the Pi-hole dashboard) from your browser.

- **The Fix:** I figured out that using the actual IP address (`127.0.0.1/admin`) instead of just `localhost` made the connection much more stable while the VPN was active. It was a good lesson in how VPN drivers can mess with local network "handshakes."

## **2.3 Locking in the Static IP**

Before I move this to other devices like my phone or TV, I realized I had a "ticking time bomb." Most routers change your laptop’s IP address every now and then. If my laptop's IP changed, all my other devices would be looking for a DNS server that "moved."

- **The Action:** I switched my Windows network settings from "Automatic" to **Manual**. I hard-coded my laptop’s current IP, set the Subnet to `255.255.255.0`, and pointed the Gateway to my router.
- **The Result:** My laptop now has a permanent "home" on the network, which is essential if it’s going to be the "brain" for the whole house.

---
# **Phase 3: Hardening the Resolver with Threat Intelligence**

## **3.1 Beyond Ad-Blocking**

After getting the traffic to flow, I realized that just blocking basic ads wasn't enough for a real security lab. I wanted to turn the **DNS Resolver** into a genuine security tool. The goal shifted from "hiding annoying banners" to "protecting the network from actual threats" like phishing sites, malware trackers, and "phone-home" telemetry from IoT devices (like Smart TVs).

## **3.2 Ingesting OSINT Feeds**

To do this, I needed better data. I turned to **Firebog.net**, which is a goldmine for **OSINT (Open Source Intelligence)**. These are lists of "known-bad" domains maintained by security researchers.

I decided to go with a "Conservative but Effective" approach. I added the **"Ticked" lists** (which are high-quality and rarely break legitimate sites) and some **Malicious/Adult** lists to keep the network clean.

- **The Process:** I copied the raw URLs for these feeds and added them to the "Adlists" section of the dashboard.
- **The "Gravity" Update:** Adding the lists wasn't enough; I had to tell the Resolver to actually go out and fetch the data. I ran the **Gravity Update**, and it was pretty satisfying to watch my blocklist jump from a few thousand domains to hundreds of thousands of entries.

## **3.3 Verifying the Sentinel**

Once the lists were updated, I had to make sure the "Sinkhole" was actually catching things. I ran some tests by visiting sites that I knew had heavy tracking.

![[Pi-Hole Query Response.png]]
_This is the moment the lab went live. You can see the 'Total Queries' spiking as my devices started hitting the Resolver, and the 'Queries Blocked' percentage showing exactly how much junk was being filtered out in real-time._

## **3.4 The YouTube Realization**

One big thing I learned during testing is that DNS-level blocking has its limits. I noticed YouTube ads were still getting through on my phone.

- **The Lesson:** I dug into the docs and realized YouTube delivers ads from the same domains as the videos. If you block the ad, you block the video.
- **The Pivot:** Instead of getting hung up on it, I accepted that DNS is for **infrastructure-level security**, while things like browser extensions are for **content-level filtering**. It was a great lesson in the "Layers of Defense" concept—no single tool can do everything.

---
# **Phase 4: Establishing the SOC Telemetry Pipeline**

## **4.1 The Goal: Enterprise Visibility**

While the dashboard on the Resolver is great for a quick glance, real security work happens in a **SIEM (Security Information and Event Management)** tool. For this lab, I chose **Splunk**. The goal was to take every "Blocked" event happening on my network and pipe it into Splunk so I could build long-term reports, track device behavior, and see exactly what the "Sentinel" was stopping over time.

## **4.2 Prepping the "Ear" (Splunk Ingestion)**

Before I could send data, I had to make sure Splunk was ready to listen.

- **The Config:** I went into Splunk's settings and opened a **TCP Receiver** on port **9997**. In the industry, this is the standard "handshake" port where Splunk waits for data to arrive from other machines.
- **The Brains:** I installed the **Pi-hole App and Add-on for Splunk**. Without these, Splunk would just see a wall of text. These apps act as a "translator" that tells Splunk, _"This part of the log is the IP address, and this part is the malicious domain."_

## **4.3 Mapping the Logs (Docker to Windows)**

Because my Resolver is running inside a Docker container, the logs are technically "trapped" inside that virtual space. To get them to Splunk, I used **Docker Volumes**.

- **The Logic:** I mapped the internal log path (`/etc/pihole/pihole.log`) to a folder on my actual Windows machine.
- **The Advantage:** This allows a **Splunk Universal Forwarder** running on my laptop to "watch" the file in real-time. Every time the Resolver blocks a tracking domain from my phone or TV, that line of text is instantly mirrored to my Windows folder and then "shot" over to Splunk.

## **Phase 4 Troubleshooting**

 **Entry: Resolving Log Path Discrepancies in Pi-hole v6**
- **Challenge:** `pihole.log` was not present in the initial volume mapping.
- **Root Cause:** Version 6 architecture stores active logs in `/var/log/pihole/` rather than `/etc/pihole/`.
 - **Resolution:** Modified `docker-compose.yml` to include a secondary bind mount for `/var/log/pihole`.
 - **Tooling:** Utilized `services.msc` to cycle the Splunk Universal Forwarder service, forcing a re-read of the modified `inputs.conf`.


---
# **Phase 4: SIEM Integration & App Optimization**

## **4.1 The Objective: Professional Visibility**

The final technical hurdle of Project Net-Filter was bridging the gap between my containerized DNS node and my **Splunk SIEM**. The goal was to move beyond simple ad-blocking and enter the realm of **Security Analytics**. I wanted a centralized "Command Center" where I could audit network-wide threats and visualize exactly what the Sinkhole was doing in real-time.

## **4.2 Engineering the Data Pipeline**

After resolving a volume-mapping issue to expose the logs (moving from `/etc/` to `/var/log/`), I utilized the **Splunk Universal Forwarder** as my "messenger."

To ensure the new configuration was active, I managed the system-level background processes.

<p align="center">
  <img src="images/Restarting%20the%20SplunkForwarder%20from%20services.msc.png" alt="Restarting the SplunkForwarder from services.msc" width="500">
</p>

_Validation of the SplunkForwarder service status within services.msc. This step was critical to ensure the Universal Forwarder was running with the updated 'inputs.conf' settings._

## **4.3 The "Proof of Life" Search**

With the forwarder live, I needed to verify that the raw data was actually reaching the indexer. I performed a targeted search in Splunk to confirm the handshake between the Docker container and the SIEM was successful.

<p align="center">
  <img src="images/Getting%20Splunk%20Logs%20from%20Pi-Hole.png" alt="Getting Splunk Logs from Pi-Hole" width="500">
</p>

_Raw telemetry validation using the SPL query 'index=main sourcetype="pihole:log"'.Seeing these initial log entries confirmed that the data pipeline was 100% functional._

## **4.4 Dashboarding & Data Visualization**

The final "Victory Moment" was transforming raw text into actionable intelligence. I installed the **xPi-Hole Analytics** app and re-mapped the internal macros to point to my custom `main` index.

<p align="center">
  <img src="images/Splunk%20Logs%20from%20the%20App%20Veiw.png" alt="Splunk Logs from the App Veiw" width="500">
</p>

_The final SOC Dashboard in the xPi-Hole app. This visualization captures the first 83 queries processed by the lab, providing a high-level view of permitted vs. blocked traffic across the network._

### Summary of Success
By the end of Phase 4, Project Net-Filter evolved from a single container into a full-stack security monitoring solution. This phase demonstrated my ability to manage Windows services, troubleshoot Docker volumes, and configure enterprise-grade analytics software to solve real-world visibility gaps.

## **4.5: Data Ingestion & Splunk Integration**

> **Objective:** Establish a persistent data pipeline between the Pi-hole DNS resolver and the Splunk Enterprise SIEM.

- **Configuration:** Created a `docker-compose.yml` volume mapping between the container path `/var/log/pihole/pihole.log` and the host directory `./var-log-pihole`.
- **Splunk Setup:** Configured a **Local File Monitor** in Splunk to "tail" the `pihole.log` file in real-time.
- **Validation:** Confirmed that DNS query telemetry is successfully traversing the Docker-to-Host bridge.

---
# **Phase 5: Performance Validation & Environmental Constraints**

## **5.1 The Objective: Stress-Testing the Pipeline**

The goal of Phase 5 was to perform a "Full-Stack" validation. I wanted to trigger a DNS request on a client machine and watch it traverse the Pi-hole engine, generate a log entry, and trigger an alert in Splunk. This phase tested the resilience of the lab against real-world networking hurdles.

## **5.2 Technical Challenges & "The Windows Bouncer"**

During testing, I encountered a significant hurdle: **Port Contention**. Windows often reserves Port 53 for internal services, creating a "Bouncer" effect that prevents Docker from receiving external DNS queries.

- **The Resolution:** I pivoted to a **"Side-Door" Validation**. By mapping the lab to Port **5353** and utilizing the internal container IP, I successfully bypassed the OS restrictions to prove the FTL (Faster Than Light) engine was operational.
- **Version 6 Adaptation:** Navigated the updated Pi-hole v6 architecture by injecting `FTLCONF` environment variables directly into the YAML manifest, forcing the resolver to accept traffic from the virtual bridge.

## **5.3 Final Telemetry Confirmation**

Even with host-level networking restrictions, the "Plumbing" of the lab remained robust. By inspecting the local directory, I confirmed that the Docker volume was successfully persisting telemetry data from the container to the Windows host for Splunk to ingest.

<p align="center">
  <img src="images/Pi-Hole%20log%20Logs.png" alt="pihole.log Logs" width="500">
</p>

_Pi-hole logs visible in Windows File Explorer or Notepad showing query data_

## **5.4 Project Conclusion: Infrastructure Logic**

While the local Windows environment presented unique routing challenges for a standard `nslookup`, the **Infrastructure-as-Code (IaC)** is verified. The lab successfully demonstrates the complete "Collect-Process-Monitor" lifecycle used in enterprise SOC environments. The project concludes with a fully functional DNS sinkhole, a hardened logging pipeline, and a ready-to-scale SIEM integration.

---
