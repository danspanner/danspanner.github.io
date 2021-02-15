---
layout: post
title: Password cracking in 2021 part two
---

With my newly built GPU backed password cracking machine, I was ready to sort out some password hashes. 

Excited, I spun up a Windows 2019 server instance and grabbed [mimikatz](https://github.com/gentilkiwi/mimikatz), which was immediately nuked by Windows Defender. So I excluded the Downloads directory of my instance, and tried again.

I was all set to grab some password hashes.

Unfortunately, all I had was the actual SAM and SECURITY file from the locked out Windows server itself. I had managed to do something quite sneaky- in Openstack, the instance was built on a Windows Server 2016 image. I converted the running image to a volume, and then mounted this volume into a Linux instance, thus allowing me to poke around in its file system.

I navigated to `/mnt/c/Windows/system32/config` and scp'd the two files to my dev server, then put them in a directory by themselves and published it as a web page with python-

```
cd ~/share-folder
python -m SimpleHTTPServer 5409
```

Python is super handy in situations like this, where you have a bunch of instances all running together in a VPC somewhere, but they're all on the same subnet. Like any good cloud noob, I have a default security group that allows all TCP for ipv4, the port is chosen largely at random.

On my mimikatz Windows server, I opened a web browser and downloaded the two files. I pointed the mimikatz binary at the two files and got a result almost immediately. Fantastic! Except...it was for the administrator on the local machine (that is, mimikatz\Administrator). Which is not great, since I want the Administrator from the SAM file I copied over.

A round of furious googling ensued, and I noticed that all the guides I was reading for mimikatz made the assumption you were on the server you wanted to crack the password for. After a little more digging, I found a few guides on how to point mimikatz at files- the downside was, these guides expected you to have exported the Windows registry with {{reg save}}.

As it turns out, a Windows registry is basically a binary file that gets executed by the operating system. So the files you find on disk are largely unreadable by tools like mimikatz. The accepted method for getting registry _out_ is to use `reg save`- in fact, despite my googling, I could not find an acceptable way to convert a SAM binary file to something I could pass to mimikatz.

I tried a variety of tools, including [reg-ripper](https://github.com/keydet89/RegRipper3.0), [chntpwd](http://www.chntpw.com/) and [hivexsh](https://libguestfs.org/hivexsh.1.html). I can't remember what mix of secret ingredients finally got me a hash, but I eventually got the hash out of the system. It looked like this.

`31d6cfe0d16ae931b73c59d7e0c089c0`

Every time I fed that hash to john the ripper, it would crack almost instantly and tell me the password was blank.

Then I read [this](https://security.stackexchange.com/questions/157922/how-are-windows-10-hashes-stored-if-the-account-is-setup-using-a-microsoft-accou#158174). Long story short, in more recent versions of Windows, Microsoft changed the way the SAM file is structured so a lot of tools no longer find the actual hash and instead report an empty hash. Super annoying, and I wasted a lot of time chasing my tail trying to figure out if I was grabbing the right files, or using the right tool, or the right command line switches.

Eventually, a friend of mine suggested I try [secretsdump](https://github.com/SecureAuthCorp/impacket/blob/master/examples/secretsdump.py). This brilliant bit of kit was exactly what I needed, although from the looks of it, I would need one more file- the `SECURITY` hive. So I copied it over with the rest of them.

From my Linux instance, I cloned the impacket repo, and ran the python script to make the magic happen.

```
cd impacket/examples
python3 secretsdump.py -sam ~/SAM -system ~/SYSTEM -security ~/SECURITY LOCAL
```

Immediately, I got a list of local accounts and their password hashes. Unfortunately, I can't show you the output, but I copied and pasted the hash section into a text file on my john the ripper server, and then pointed john at my hashes-

```
./john --format=NT-opencl hash.txt --fork=32
```

This was an interesting journey to go through, with a lot of very frustrating dead ends that I followed, but I'm glad I went through it. In the end, my team found another way into the server that (unfortunately) did not involve anything anywhere near as cool as cracking a password hash. But at the end of the day, it's the story that counts.