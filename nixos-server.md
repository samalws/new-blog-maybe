# My Nixos server setup: a walkthrough
Bla bla bla intro
First off, heres' why I chose Nixos as my distro, and why it's objectively better than any of your objectively bad distros.
## Nixos confirmed best distro for sysadmin
When I started setting up my server using Nixos, I was expecting it to be just as painful as
any normal Linux setup, or maybe a little easier.
To my surprise, it turned out to be really easy and satisfying.
I feel like sysadmin might be one of Nixos' biggest strong suits.
It solves basically all the things about server hosting that used to annoy me:
- **Editing a million config files**: In Nixos, you can specify all your config in the Nix language, and it'll auto-generate the config files in the right formats for you. Also, all your config can be in one single file (`configuration.nix`) rather than spread out all over the place.
- **Writing your own systemd daemons**: This is kinda a subset of "editing a million config files" but I think it deserves its own bullet point. Writing systemd daemons on your own sucks. In Nixos, it's a few lines of Nix code just like anything else. *(Yes, you have to use systemd. Yes, systemd kinda sucks, I'll give you that)*
- **Messing something up and not being able to undo it**: lskdjf atomic,
- **Permissions / having to be root to install everything**: Let's say you want to use a program, but you don't need to have it permanently installed, you're scared installing it globally will mess something up, or you don't have root access. In Debian etc. you'd be screwed. In Nixos, you can just run `nix-shell -p [your program]`.
## Nixos install speedrun any%
Now that I've done my duty as professional Nixos shill, let's go through how to
get an installation going that has most of the same features as mine.
The installation includes a FTP, SSH, git, and HTTP server.
(We're going to put off installing an email server until later, when we get DNS set up). TODO maybe not
Here(link to "Nixos server any% " video goes here)'s a video to go along with the text instructions.
Anyways, on with the tutorial:
- **Part 1: Vultr stuff**
- Sign up for Vultr, a cloud computing service. It's pretty cheap: my server costs $3.50/month. Feel free use something else if you're willing to adapt my instructions a bit. If you *do* choose to use Vultr, you should use this(TODO link) referral link, so that I get free money and you get $100 of cloud computing credit.
- On the "products" page, click the big plus sign, and then "deploy new server".
- Bla bla bla filling out options
- **Part 2: Installing from the livecd**
- test3
- **Part 3: Booting into your server, for real this time**
At this point, (stuff you should have)
## DNS Stuff: more annoying than you'd think
I lied when I said that your server is entirely determined by your Nixos config file.
Something something not shilling Google.
## SSL Certs: less annoying than you'd think
When you search up how to get an SSL cert at first, it seems like some long and complicated process that
involves you having to pay money.
In reality, it's dead simple, pretty much just running one command.
## Nix config, but I actually explain it instead of glossing it over like I did before
## Adding the email server TODO maybe earlier
## Trying out your email server TODO this and the previous should be mutually exclusive, also should maybe come before "nix config, but i actually explain it"
Now that that's over with, our email server should work.
You can test your server with Thunderbird, which Just Works:tm: for the most part.
I don't use Thunderbird for email anymore, but it's a good starting point
before you go full Linux neckbeard mode and use command line email clients (which you should!)
## Trying out FTP
## Setting up some git repos
