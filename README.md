
DCN320 — Lab 2B: Packet Sniffing & Spoofing
Student: REPLACE_WITH_YOUR_NAME
Date: Run date +%F inside the SEED VM and enter the date.
Environment: SEED Ubuntu 20.04 on VirtualBox

Important: Run commands one block at a time. Do not run the whole README as a script. Only test sniffing/spoofing inside your isolated, authorized SEED lab network. This README covers verified Docker tutorial commands and a preparatory workflow for Task 3. The exact Section 3.1–3.4 questions/code must be checked against your instructor's PDF, which was not attached here.

Task 1 — Verify the SEED VM
VirtualBox: VM type Ubuntu (64-bit); existing SEED-Ubuntu20.04.vdi; EFI disabled; VMSVGA; video memory 128 MB; 2 CPUs; 4096 MB RAM.
In VirtualBox create a NAT Network and attach Adapter 1 to it, as instructed by the instructor.
Boot VM. Login: user seed, password dees.
Open terminal and run:
```bash
whoami
hostname
date
ip -br addr
ping -c 4 google.com
```
Show your name and date (replace the placeholder):
```
printf 'Student: %s | Date: %s\n' 'REPLACE_WITH_YOUR_NAME' "$(date +%F)"
```
SCREENSHOT T1-1 (REQUIRED): SEED VM desktop and terminal showing your name and date.
SCREENSHOT T1-2 (RECOMMENDED): VirtualBox NAT Network settings.

Task 2 — Official SEED Docker Tutorial
Reference: https://github.com/seed-labs/seed-labs/blob/master/manuals/docker/docker.md

2.1 Check Docker
Run on SEED VM host:
```
sudo systemctl status docker --no-pager
sudo docker --version
sudo docker ps -a
```
If Docker is installed but stopped:
```
sudo systemctl start docker
```
2.2 Create the tutorial working directory and owner.txt
```
mkdir -p ~/DCN320-Lab2B/docker-tutorial
cd ~/DCN320-Lab2B/docker-tutorial
printf 'Student: %s\nDate: %s\n' 'REPLACE_WITH_YOUR_NAME' "$(date +%F)" > owner.txt
cat owner.txt
```
2.3 Create the corrected Dockerfile
Copy this entire block into the SEED VM terminal:
```
cat > Dockerfile <<'EOF_DOCKER'
FROM ubuntu:20.04
ARG DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
    && apt-get -y install \
       iputils-ping \
       iproute2 \
       net-tools \
       dnsutils \
       mtr-tiny \
       nano \
    && apt-get clean
# Corrected COPY: local file owner.txt goes into the container.
COPY owner.txt /owner.txt
CMD ["/bin/bash"]
EOF_DOCKER
cat Dockerfile
```
SCREENSHOT T2-1 (REQUIRED): Terminal showing complete corrected Dockerfile, especially COPY owner.txt /owner.txt.

2.4 Create network and build the image
```
sudo docker network create --subnet=172.20.0.0/24 seednetwork
sudo docker network ls
sudo docker build -t seedhost .
```
If seednetwork already exists, do not recreate it. If the subnet conflicts with another Docker network, inspect sudo docker network ls and resolve the conflict rather than deleting unrelated networks.

SCREENSHOT T2-2 (REQUIRED): Successful docker build output.

2.5 Run container and verify owner.txt
```
sudo docker run --name dcn320-seed-tutorial --net seednetwork --ip 172.20.0.5 --rm -it seedhost
```
You are now INSIDE the container (prompt changes). Run:
```
whoami
hostname
ip addr
cat /owner.txt
ping -c 3 172.20.0.1
```
SCREENSHOT T2-3 (REQUIRED): cat /owner.txt output from INSIDE the running container, showing your name and date.
SCREENSHOT T2-4 (REQUIRED): Successful container commands (e.g. ip addr, ping).

Exit container:
```
exit
```
Back on SEED VM host, inspect Docker:
```
sudo docker ps
sudo docker ps -a
sudo docker images
sudo docker network ls
```
SCREENSHOT T2-5 (RECOMMENDED): Docker image/network lists and container completion.

Mandatory cleanup between Task 2 and Task 3
Run on SEED VM host, after taking all Task 2 screenshots:
```
sudo docker ps -a
sudo docker network ls
sudo docker rm -f dcn320-seed-tutorial 2>/dev/null || true
sudo docker network rm seednetwork
sudo docker ps -a
sudo docker network ls
```
The tutorial container uses --rm, so it may already be gone. Do not run docker system prune -a or remove all containers: other labs might be affected. The seedhost image can remain because it does not occupy ports.

SCREENSHOT CLEANUP (RECOMMENDED): docker ps -a and docker network ls after cleanup.

Task 3 — Sniffing & Spoofing Lab (through Section 3.4)
Official lab landing page: https://seedsecuritylabs.org/Labs_20.04/Networking/Sniffing_Spoofing/

Required instructor materials: N04_Sniffing_Spoofing.pptx and Lab2B_Sniffing_Spoofing.pdf. To avoid mismatched steps, refer to the actual PDF for each original requirement and question. The commands below are setup and diagnostics, not a claim that Sections 3.1–3.4 are complete.

3.0 Download and start Labsetup
Download Labsetup.zip from the official lab webpage into the SEED VM's Downloads directory.
Use the actual filename and path from your Downloads folder:
```
cd ~/Downloads
ls -lh
unzip Labsetup.zip
cd Labsetup
ls -la
```
If Labsetup is in another location, use find ~/Downloads -maxdepth 3 -name docker-compose.yml to locate it.

Inspect the supplied compose configuration:
```
cat docker-compose.yml
sudo docker-compose build
sudo docker-compose up -d
sudo docker ps
```
If your VM uses the modern Compose plugin instead, substitute sudo docker compose for sudo docker-compose.

SCREENSHOT T3-SETUP (REQUIRED FOR REPORT EVIDENCE): Successful containers running with sudo docker ps.

Discover interfaces and container addresses
```
ip -br addr
ip route
sudo docker ps --format 'table {{.Names}}\t{{.ID}}\t{{.Status}}'
sudo docker network ls
```
Use the actual container names and IP addresses shown in your VM. Do not assume a specific interface name.

Packet-capture checks
Start Wireshark on the SEED VM using the application menu (or wireshark in a terminal). Choose the interface connected to the SEED lab network, not the internet NAT interface. A simple display filter for ICMP traffic is:
```
icmp
```
Generate test ICMP traffic between your lab containers using the addresses listed in the supplied compose file. Capture a packet, select it, and show the Ethernet/IP/ICMP fields.

SCREENSHOT T3-CAPTURE: Wireshark capture with packet list, selected packet details, and visible IP addresses.

Section-by-section report checklist (fill from your PDF)
Section 1.0: Copy exact title and original requirements; show setup and requested evidence.
Section 2.0: Copy exact title and original requirements; show environment commands/results.
Section 3.1: Copy exact subsection title and instructions; include code, outputs, capture, and answer to its questions.
Section 3.2: Copy exact subsection title and instructions; include code, outputs, capture, and answer to its questions.
Section 3.3: Copy exact subsection title and instructions; include code, outputs, capture, and answer to its questions.
Section 3.4: Copy exact subsection title and instructions; include code, outputs, capture, and answer to its questions.
STOP after Section 3.4. Do not include later lab sections unless your instructor requests them.

When finished
```
cd ~/Downloads/Labsetup
sudo docker-compose down
sudo docker ps -a
```
If the compose project is located elsewhere, cd to the folder containing docker-compose.yml before down.

Report reminders
Preserve all original section titles and requirements from the instructor's PDF.
Paste the actual terminal output, not sample output.
Add screenshots immediately after each relevant section or in a labeled appendix.
Include code snippets and short, simple-English explanations.
Verify screenshots show the correct lab VM, command, and result.

```
#!/usr/bin/env python3

from scapy.all import sniff, ICMP

def print_packet(pkt):
    if pkt.haslayer(ICMP):
        pkt.show()

sniff(
    iface="br-6f5122c1b8fa",
    filter="icmp",
    prn=print_packet,
    store=False
)
```




```
cd ~/Desktop
nano filter1.py
```
```
#!/usr/bin/env python3
from scapy.all import *

def print_pkt(pkt):
    pkt.show()

pkt = sniff(
    iface="br-6f5122c1b8fa",
    filter="icmp",
    prn=print_pkt
)
```


```
chmod a+x filter1.py
sudo ./filter1.py
```

```
sudo docker exec hostA-10.9.0.5 ping -c 5 10.9.0.6
```


```
sudo docker exec hostA-10.9.0.5 ping -c 5 10.9.0.6
```


```
cd ~/Desktop
nano filter2.py



#!/usr/bin/env python3
from scapy.all import *

def print_pkt(pkt):
    pkt.show()

pkt = sniff(
    iface="br-6f5122c1b8fa",
    filter="tcp and src host 10.9.0.5 and dst port 23",
    prn=print_pkt
)



chmod a+x filter2.py
sudo ./filter2.py



sudo docker exec -it hostA-10.9.0.5 /bin/bash


telnet 10.9.0.6 23

apt-get update
apt-get install -y telnet
telnet 10.9.0.6 23




cd ~/Desktop
nano filter3.py

#!/usr/bin/env python3
from scapy.all import *

def print_pkt(pkt):
    pkt.show()

pkt = sniff(
    iface="br-6f5122c1b8fa",
    filter="net 128.230.0.0/16",
    prn=print_pkt
)

chmod a+x filter3.py
sudo ./filter3.py


sudo docker exec -it hostA-10.9.0.5 /bin/bash


ping -c 5 -W 2 128.230.0.1


cd ~/Desktop
nano spoof.py


#!/usr/bin/env python3
from scapy.all import *

# Create an IP packet with a fake source IP
a = IP()
a.dst = '10.9.0.5'
a.src = '1.2.3.4'

# Create an ICMP Echo Request
b = ICMP()

# Combine and send the packet
p = a/b
send(p)

chmod a+x spoof.py
sudo ./spoof.py




cd ~/Desktop
nano traceroute.py




#!/usr/bin/env python3
from scapy.all import *

target = '8.8.8.8'

for ttl in range(1, 16):

    # Create IP packet with increasing TTL
    a = IP(dst=target, ttl=ttl)
    b = ICMP()

    # Send packet and wait for a reply
    reply = sr1(a/b, verbose=0, timeout=2)

    if reply is None:
        print(f"{ttl}: * * * (No response)")
    else:
        # Print the IP of the router that replied
        print(f"{ttl}: {reply.src}")

        # Stop when destination sends Echo Reply
        if reply.haslayer(ICMP) and reply[ICMP].type == 0:
            print("Reached destination!")
            break


chmod a+x traceroute.py
sudo ./traceroute.py





Step 1 — إنشاء الملف
افتحي Terminal العادي في Ubuntu واكتبي:
cd ~/Desktop
nano sniff_spoof.py


انسخي الكود كامل:
#!/usr/bin/env python3from scapy.all import *def spoof_pkt(pkt):    if IP in pkt and ICMP in pkt and pkt[ICMP].type == 8:        src = pkt[IP].src        dst = pkt[IP].dst        print(f"Sniffed ICMP request: {src} -> {dst}")        ip = IP(src=dst, dst=src)        icmp = ICMP(            type=0,            id=pkt[ICMP].id,            seq=pkt[ICMP].seq        )        reply = ip / icmp / bytes(pkt[ICMP].payload)        send(reply, verbose=0)        print(f"Spoofed reply sent to {src} from {dst}")sniff(    iface="br-6f5122c1b8fa",    filter="icmp",    prn=spoof_pkt,    store=False)



احفظي بـ Ctrl+O ثم Enter ثم Ctrl+X.
Step 2 — تشغيل البرنامج
في نفس Terminal:
chmod a+x sniff_spoof.py
sudo ./sniff_spoof.py


خليه شغال ولا تقفلينه.
Step 3 — افتحي Terminal ثاني وادخلي Host A
sudo docker exec -it hostA-10.9.0.5 /bin/bash


المفروض يتغير السطر إلى root@...:/#.
Step 4 — الاختبار الأول: 1.2.3.4
داخل Host A:
ping -c 4 -W 2 1.2.3.4


ارجعي للـTerminal الأول وشوفي هل ظهرت رسائل Sniffed ICMP request وSpoofed reply sent.

📸 Screenshot 1 — Ping 1.2.3.4: صوري نتيجة الـPing، وصورة ثانية للـSniffer إذا ظهرت الرسائل.
Step 5 — الاختبار الثاني: 10.9.0.99
في نفس Terminal الخاص بـHost A:
ping -c 4 -W 2 10.9.0.99


هذا عنوان داخل نفس الشبكة لكنه غير مستخدم في اللاب.

ممكن يظهر:
Destination Host Unreachable


وهذا متوقع لأن Host A يحاول معرفة عنوان MAC باستخدام ARP، وإذا ما فيه جهاز يرد على ARP، ما يقدر يرسل ICMP Echo Request.

📸 Screenshot 2 — Ping 10.9.0.99: صوري النتيجة حتى لو فشلت.
Step 6 — الاختبار الثالث: 8.8.8.8
داخل Host A:
ping -c 4 -W 2 8.8.8.8


ارجعي للـTerminal الأول وراقبي الرسائل.

قد تشوفين ردود حقيقية أو مزيفة، حسب إعدادات الشبكة واتصال الإنترنت.

📸 Screenshot 3 — Ping 8.8.8.8: صوري نتيجة الـPing والرسائل اللي ظهرت في برنامج sniff_spoof.py.
Step 7 — اختبار Routing وARP
داخل Host A نفذي هذه الأوامر:
ip route
ip route get 1.2.3.4
ip route get 10.9.0.99
ip route get 8.8.8.8
ip neigh show



icmp && ip.addr == 1.2.3.4

cd ~/Desktop
sudo ./spoof.py
```
```
#!/usr/bin/env python3
from scapy.all import *

def spoof_pkt(pkt):
    if IP in pkt and ICMP in pkt and pkt[ICMP].type == 8:
        src = pkt[IP].src
        dst = pkt[IP].dst

        print(f"Sniffed ICMP request: {src} -> {dst}")

        ip = IP(src=dst, dst=src)
        icmp = ICMP(
            type=0,
            id=pkt[ICMP].id,
            seq=pkt[ICMP].seq
        )

        reply = ip / icmp / bytes(pkt[ICMP].payload)

        send(reply, verbose=0)
        print(f"Spoofed reply sent to {src} from {dst}")

sniff(
    iface="br-6f5122c1b8fa",
    filter="icmp",
    prn=spoof_pkt,
    store=False
)
```

