# Recon

I have broken up this recon guide into manual and a more automated appraoch. Special thanks to @jhaddix and @tomnomnom, for their guides and tools!

References: 
    - [The Bug Hunter's Methodology by @jhaddix](https://www.youtube.com/watch?v=gIz_yn0Uvb8&ab_channel=RedTeamVillage)
    - [Bug Bounty Program with @TomNomNom and @Nahamsec](https://www.youtube.com/watch?v=SYExiynPEKM&ab_channel=Nahamsec)


## Manual Recon

This section was mostly currated from @jhaddix's methodology.

### Finding Seeds/Roots

#### Find ASN

Manual:
http://bgp.he.net


#### Ad/Analytic Relationships
Find popular or less popular pages. Can see what ad tools were used on the site and related sites.

builtwith.com

### Finding Subdomains

#### Linked and JS Discovery

1. Spider the site using burp.
	1. Set a burp target scope
	2. Select all sites in "Site map" to spider.
	3. To get the sites out of burp: 
		1. Right click selected hosts
		2. Go to "Engagment Tools" -> "Analyze target"
		3. Save report as html file
		4. Copy hosts form the "Target" section

#### Subdomain scraping

1. google search removing things you already know.
	1. `site:twitch.tv -www.twitch.tv`
	2. `site:twitch.tv -www.twitch.tv -watch.twitch.tv`
2. Amass will automate this for you.
3. Subfinder https://github.com/projectdiscovery/subfinder
4. github-subdomains.py https://github.com/gwen001/github-search/blob/master/github-subdomains.py
5. shosubgo https://github.com/incogbyte/shosubgo
6. Cloud Ranges - scans AWS, GCP and Azure for SSL sites within certificates. (Sam Erb automated this.)

#### Subdomain bruteforcing

Searching is only as good as the wordlist you provide.
https://github.com/assetnote/commonspeak2
https://github.com/assetnote/commonspeak2-wordlists/tree/master/subdomains

1. `amass enum -brute -d twitch.tv -src`
	1. Has a built in list but you can supply your own.
2. `amas enum -brute -d twitch.tv -rf resolvers.txt -w bruteforce.list`
	1. resolvers.txt would be a list of DNS servers.
3. shuffledns (wrapper around amass) https://github.com/projectdiscovery/shuffledns


### Manual - Other

#### Favicon Analysis
1. favfreak: https://github.com/devanshbatham/FavFreak
	1. Get the hash of the favicon and use Shodan to search with that hash.

#### Port Analysis
1. `masscan -p1-65535 -iL $ipFIle --max-rate 1800 -oG masscan.log`
2. Sends output to nmap service scan -oG
3. Sends output to brutespray

#### Github dorking (manual)

#### Screenshotting

To help prioritize work.

1. Aquatone
2. httpscreenshot
3. eyewitness

#### Subdomain takeover
https://github.com/EdOverflow/can-i-take-over-xyz

## Automated Recon

This section was mostly currated from @tomnomnom's methodology/toolset.
During the walk through Tom used shopify as an example, make sure to subsitute below where appropriate. 

### Prerequisites on Kali
```bash
sudo apt-get install go
go get -u github.com/tomnomnom/assetfinder
go get -u github.com/tomnomnom/anew
go get -u github.com/tomnomnom/httprobe
go get -u github.com/tomnomnom/fff
```

Your `~/go/bin/` should look like this now.
```bash
ls      
anew  assetfinder  httprobe  fff
```

#### Install findomain
```bash
cd /tmp/
wget https://github.com/findomain/findomain/releases/latest/download/findomain-linux
chmod +x findomain-linux
sudo cp findomain-linux /usr/bin/findomain
```

### Recon - Start
```bash
# Create and go into our workspace for this session.
mkdir -p ~/recon/shopify; cd ~/recon/shopify

# create a file with the 2 noted wildcards in hacker1.
cat wildcards                 
shopifykloud.com
shopify.com

# Pipe wildcards to `assetfinder` then to `anew` which keeps unique strings.
cat wildcards| assetfinder --subs-only | anew domains
```
 

```bash
cat domains | httprobe -c 80 --prefer-https
```

```bash
findomain -f wildcards | tee -a findomain.out
```

Tom created a new file with some of the domains from the above command called from-findomain.
Simply looked for all sites that ended with a .com, could use a regex.
```bash
cat findomain.out | grep .com | grep -v '>' > from-findomain
```

I ran `grep -v .com findomain.out` to verify I did not miss any domains

Cool.. next command ...  to further build out availible hosts.
```bash
cat from-findomain | anew domains; cat domains | httprobe -c 50 | anew hosts
```

Query and save each page.
```bash
cat hosts | fff -d 1 -S -o roots
```

Branching off from tomnomnom. 
Send each page to burp or the like proxy.
```bash
cat hosts | fff -d 50 -x http://localhost:8080
```
