# Install Keitaro on VPS: Which Specs You Need by Click Volume, How to Answer the Installer's Prompts, and What Breaks After Setup

Keitaro runs on a clean KVM server with CentOS 9 Stream or CentOS 10 Stream, at least 4 GB of RAM, 20 GB of SSD, no hosting control panel, and no NAT between the box and the internet. Meet those and the install is one command:

bash
curl https://files.keitaro.io/scripts/releases/stable/kctl.sh | bash -s -- install


The work that takes longer than the install is everything around it: buying a plan sized to the traffic you actually expect, answering the installer's nine prompts correctly, and handling domains, SSL, firewall ports and extra-IP licensing once the tracker is live. That's what this guide covers.

> **Quick picks**
>
> - **Under 100,000 clicks a day:** 2 vCPU / 4 GB. Ava Hosting's VPS Basic starts at €10.00/month for 2 vCPU, 4 GB RAM and 50 GB NVMe, which is Keitaro's stated minimum with room for the database to grow.
> - **100,000 to 500,000 clicks a day:** 4 vCPU / 8 GB. VPS Power, from €20.00/month, 90 GB NVMe.
> - **500,000 to 1,000,000 clicks a day:** 16 GB RAM. VPS Ultra, from €40.00/month. It's the largest plan Ava lists, so past roughly a million clicks a day you'll be sizing elsewhere.
>
> 👉 [Check current Ava Hosting VPS pricing](https://my.ava.hosting/aff.php?aff=15562)

## Ava Hosting VPS plans compared

Ava's VPS line runs from a €11/year entry box to a 16 GB machine, all on NVMe storage with root access.

| Plan | RAM / vCPU | NVMe | Price | Order |
| --- | --- | --- | --- | --- |
| VPS Nano | 512 MB / 1 | 20 GB | from €11.00/year | [VPS Nano](https://my.ava.hosting/aff.php?aff=15562) |
| VPS One | 2 GB / 1 | 25 GB | from €5.00/month | [VPS One](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Basic | 4 GB / 2 | 50 GB | from €10.00/month | [VPS Basic](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Lucky | 6 GB / 2 | 70 GB | from €15.00/month | [VPS Lucky](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Power | 8 GB / 4 | 90 GB | from €20.00/month | [VPS Power](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Extra | 12 GB / 6 | 120 GB | from €30.00/month | [VPS Extra](https://my.ava.hosting/aff.php?aff=15562) |
| VPS Ultra | 16 GB / 8 | 160 GB | from €40.00/month | [VPS Ultra](https://my.ava.hosting/aff.php?aff=15562) |

Prices are in EUR and billed monthly except VPS Nano, which is annual. Every plan includes full root access, a 1 Gbps port, unlimited bandwidth, DDoS protection and weekly backups. Ava's order pages also list a parallel "Self-hosted Start" line (Start, Mini, Basic, Pro, Business, Power) at €2–€40/month with the same RAM and vCPU steps but different NVMe sizes; the tier mapping below applies to either.

Two things to settle at checkout, because Keitaro is unforgiving about both. First, the plan pages list "Control Panel" among the included features, which in VPS billing normally refers to the management panel in the client area. Keitaro needs a bare OS with no hosting panel (cPanel, Plesk, ISPmanager) layered on top, so pick the plain CentOS 9 Stream template and skip any panel add-on. Second, Keitaro requires KVM virtualization and does not run behind NAT, and Ava's product pages don't spell out the virtualization type or the IP arrangement. Confirm both with support before you pay, and make sure the plan comes with a dedicated IPv4 address rather than a shared NAT one.

## What Keitaro needs, in numbers

Keitaro's documentation sizes servers by daily click volume instead of giving one blanket minimum, which is more useful when you're deciding how much RAM to pay for.

| Daily clicks | Server type | RAM | CPU | Ava plan that fits |
| --- | --- | --- | --- | --- |
| Under 100,000 | VPS | 4 GB | 2 | VPS Basic, €10.00/month |
| 100,000 – 500,000 | VPS | 8 GB | 4 | VPS Power, €20.00/month |
| 500,000 – 1,000,000 | VPS / VDS | 16 GB | 4 | VPS Ultra, €40.00/month |
| 1,000,000 – 5,000,000 | VDS | 32 GB | 4 | none, Ava tops out at 16 GB |
| 5,000,000 – 10,000,000 | VDS | 64 GB | 8 | none, Ava tops out at 16 GB |

Beyond the RAM and CPU numbers, the mandatory list is short and absolute:

- A clean server with no control panel installed
- CentOS 9 Stream or CentOS 10 Stream
- KVM virtualization
- 20 GB SSD or more
- 4 GB RAM minimum
- No NAT in front of the server

That last one disqualifies a whole category of budget hosting. NAT VPS plans sell cheap because several customers share one public IPv4 address, and Keitaro simply doesn't support running that way.

Keitaro's docs also note that a 2 GB / 1 CPU server will install, with a warning that performance problems may occur. Treat that as a testing box, not a production tracker. VPS One at €5.00/month is that box. VPS Nano, at 512 MB, doesn't reach the starting line at all, regardless of its €11/year price.

The CentOS requirement is where older tutorials mislead you. Keitaro's legacy help page still says "CentOS 7 and higher", while the current documentation lists only CentOS 9 Stream and CentOS 10 Stream. Use 9 or 10 Stream; CentOS 7 reached end of life and expecting an installer built for it to behave on a modern box is a waste of an afternoon.

## Picking between Ava's plans

The middle three plans don't map cleanly onto Keitaro's table, and that's the part worth thinking about.

VPS Lucky gives you 6 GB and still only 2 vCPU for €15.00/month. Extra RAM helps MariaDB and Redis, but the official spec for 100k–500k clicks a day is 8 GB *and* 4 CPU, so Lucky doesn't move you into the next tier. If you expect to cross 100,000 clicks a day within the life of the server, the extra €5.00/month for VPS Power buys the CPU cores you'll otherwise be missing.

VPS Extra goes the other direction: 6 vCPU and 12 GB for €30.00/month. CPU is above what Keitaro asks for at the 500k–1M tier, RAM is below it. It suits someone sitting around 300,000–400,000 clicks a day who wants headroom for redirects and cron jobs but can't justify doubling the RAM.

Above 1,000,000 clicks a day the table asks for 32 GB, then 64 GB. Ava's largest plan is 16 GB, so that's the ceiling of this line-up. Running multiple trackers or splitting traffic across two servers is a more practical route than hunting for a 64 GB box.

👉 [See what each Ava Hosting plan costs today](https://my.ava.hosting/aff.php?aff=15562)

## Installing Keitaro step by step

Log in over SSH as root, then run the command. On Windows use PuTTY; macOS and Linux have a terminal built in. Most cloud providers also expose a browser console for the server, which is worth keeping open in a second tab for reasons covered below.

bash
ssh root@your_server_ip
curl https://files.keitaro.io/scripts/releases/stable/kctl.sh | bash -s -- install


The script then asks a fixed sequence of questions. Answers in brackets are the defaults, and pressing Enter accepts them.

**Free Let's Encrypt SSL certificate?** Answer `yes` if the tracker will live on a domain, and the script will request a certificate with automatic renewal. Answer `no` and the admin panel stays on `http://your_server_ip/admin/` over plain HTTP.

**List your domains, comma separated.** Only appears if you said yes to SSL. Enter every domain the tracker should answer on, for example `track.example.com,go.example.com`.

**Use a database from an SQL dump?** `yes` only when you're migrating an existing tracker to a new server. For a fresh install, `no`.

**Database name, database user, database password.** Defaults are `keitaro` for the first two; the password is generated. Write it down if you change it.

**Keitaro admin name and password.** The login for the tracker itself. The name defaults to `admin`.

**Server IP address.** The public IPv4 of this machine.

**License key.** You get this from your account on keitaro.io, in the Licenses section. The installer won't finish without a valid one.

The script provisions Nginx, MariaDB, Redis and PHP, wires up PHP-FPM, installs Monit to watch those services, configures the firewall, requests the SSL certificate if you asked for one, and puts Keitaro in `/var/www/keitaro`. When it's done, open the address it prints.

If the install dies partway through, run the exact same command again. Every parameter you already entered is restored automatically, so you're not retyping the database password and license key. To see what went wrong, read the log:

bash
tail -n 1000 /var/log/keitaro/install.log


Plenty of tutorials still show a different command, `yum update -y && curl keitaro.io/install.sh > run && bash run`. That's Keitaro's older installer, and parts of its own help site still carry it. The current documentation uses the `kctl.sh` line above. If a video walkthrough doesn't match what you see on screen, that mismatch is usually why.

## After the install: domains, SSL, firewall ports, extra IPs

**Pointing a domain at the tracker.** Add an A record in the domain's DNS, `@ IN A 1.2.3.4`, with your server's IP in place of 1.2.3.4. Once it propagates, `http://newdomain.com/admin` should serve the login page.

**Adding SSL later.** If you skipped the certificate during install, you can add it afterwards:

bash
curl keitaro.io/enable-ssl.sh > run; bash run -D domain.com,domain2.com


This only works on a server that the one-click script configured in the first place. On a hand-built stack it has nothing to attach the certificate to.

**The firewall only opens four ports.** The installer closes everything except 22, 3333, 80 and 443. Port 3333 is there for one specific host's SSH setup. If your SSH daemon listens on any other port, you lose console access the moment the firewall reloads, and the fix has to happen through the provider's web terminal or rescue mode. Check your SSH port before you run the installer.

# Open ports` section and add a line:

bash
iptables -A INPUT -p tcp -m tcp --dport 3306 -j ACCEPT


Then apply it with `systemctl restart firewall`.

**Extra IP addresses cost a license each.** You can attach additional IPs to the server and run domains on them, but every IP needs its own active Keitaro license. Install using the license key for the primary IP; the additional IPs then work without further configuration. Budget for that before promising a client a dedicated IP.

**Switching the PHP engine.** To move from PHP-FPM to RoadRunner and upgrade it:

bash
curl https://keitaro.io/install.sh > run; bash run -rt upgrade


To switch back:

bash
export PHP_ENGINE=php-fpm; curl keitaro.io/install.sh | bash -s -- -rt tune


For routine OS updates, `yum update` is enough.

## FAQ

**Can I install Keitaro on a server that already runs websites or has a control panel?**
No. Keitaro's docs require a clean server and warn that running the script on a machine with existing sites or a panel has unpredictable consequences. Spin up a fresh VPS; the €10.00/month for a separate box is cheaper than repairing a broken production server.

**The installer stopped halfway. Do I start over?**
Re-run the same command. Previously entered parameters are restored, including the license key. Check `tail -n 1000 /var/log/keitaro/install.log` for the failing step before you retry, so you're not repeating the same error.

**Why did I lose SSH access right after installing?**
Your SSH port isn't 22 or 3333, and the installer's firewall dropped the connection. Use the provider's web console to get back in, then add your port to `/etc/firewall.bash` and restart the firewall service.

**Is 2 GB of RAM workable?**
Keitaro's docs say installation is possible on 2 GB / 1 CPU and add that performance issues may occur. Fine for testing redirects and the interface. Not what you want handling paid traffic.

**Do extra domains need extra licenses?**
Domains and mirrors are handled with DNS A records, and the docs describe no per-domain licensing. IP addresses are different: each additional IP requires an active license.

**Which install command is current?**
`curl https://files.keitaro.io/scripts/releases/stable/kctl.sh | bash -s -- install`. The `keitaro.io/install.sh` variant still appears on older pages and in the commands for engine switching, but the current installation guide uses `kctl.sh`.

## Before you order

Match the plan to the traffic you expect, not to the traffic you hope for. Under 100,000 clicks a day, VPS Basic at €10.00/month covers Keitaro's official minimum. Between 100,000 and 500,000, VPS Power at €20.00/month is the one that meets both the RAM and CPU figures. Half a million to a million clicks a day puts you on VPS Ultra at €40.00/month, the top of the range.

Then confirm four things before paying: KVM virtualization, a dedicated IPv4 rather than NAT, CentOS 9 Stream or CentOS 10 Stream available as an OS template, and no hosting panel on the install. Any one of them missing turns a ten-minute install into a support ticket.

👉 [Pick the Ava Hosting VPS tier that matches your click volume](https://my.ava.hosting/aff.php?aff=15562)
