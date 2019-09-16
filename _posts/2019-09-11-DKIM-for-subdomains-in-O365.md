---
layout: post
title:  "DKIM for subdomains in O365"
---

## So DKIM is pretty nice.

For everyone else, of course. It does very little to protect your own users and domain, and it's one of those "if everyone is using it, it works great!" sort of deals.

Well, I went on an adventure recently to protect my institution from the evils of mail spam, and as part of that adventure I introduced DKIM and DMARC records (SPF already being enabled when I showed up).

So I covered our MX enabled domains. Fantastic. The process was easy and relatively painless, thanks to O365. It's basically just a "Make DKIM go now" button. 

I'm not kidding. It's literally a button. 

But then, just to throw me a curveball, I got a panicked email.

_Uh oh, has DKIM broken everything?_ I thought.

Nope. Turns out, marketing material had been sent out with info@subdomain.domain.com instead of info@otherdomain.com. Can I make an email address for it?

Easy done. Five minutes work. I added the subdomain, added MX records, DMARC and SPF...

And the tickbox was gone. Well. Looks like this adventure isn't over.

So, I made some preparations. 

I added two new CNAMEs in my domains DNS-

```
selector1._domainkey.subdomain  CNAME   selector1-subdomain-domain._domainkey.officetenant
selector2._domainkey.subdomain  CNAME   selector2-subdomain-domain._domainkey.officetenant
```

and a TXT record of course

```
_dmarc.subdomain    TXT     v=DMARC1; pct=100; p=quarantine; rua=mailto:dmarc@domain; ruf=mailto:dmarc@domain;
```

All the pieces are in place, except for my tickbox. Back to the googles I suppose.

Some time passed, and I got distracted by something else. I was, in fact, poking around in powershell trying to fix a problem with our on-prem Exchange, and a light dinged. Once again, powershell to the rescue!

Turns out, you can flick the switch for your [DKIM settings in powershell]("https://docs.microsoft.com/en-us/powershell/module/exchange/antispam-antimalware/new-dkimsigningconfig?view=exchange-ps"). This is probably something that should have occured to me sooner.

Once you have a DKIM signing policy set, you can [now enable it]("https://docs.microsoft.com/en-us/powershell/module/exchange/antispam-antimalware/set-dkimsigningconfig?view=exchange-ps").
