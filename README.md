#DomainLockDown

DLD was written for the AD admin who either isn't sure what best practices to use to secure their domain controllers, or how best to secure their DA accounts (which do need handling. It's not enough to simply set them up and walk away). 

It was __not__ written to solve the world's AD security problems. Currently it's targeted towards DA accounts and DCs. If you don't patch your DCs or set your Enterprise Admin password to something silly, well, that's on you.

##Some examples of when to use DLD:

1. You have a brand new domain and want a quick security fix.
2. You want to a simple baseline check against common AD attack vectors.
3. You want to improve the security posture of an existing domain.

1 and 2 are obvious, but 3 requires explanation which I'll get into shortly. First, it is important to understand the prereqs of DLD and the methodology used for remediation.

###DISCLAIMER

If you tell it to, DLD will make changes to your AD domain that may impact how AD interacts with your users & applications. You are prompted before any changes occur. You are responsible for what you do to your domain, not me. Please please please run this in a test domain first. __You've been warned.__

###REMEDIATION METHODOLOGY

It is very important to understand how DLD improves the security of your environment (prior to running it - unless you just want to play around, in which case have fun! :-). DLD will modify the login abilities of your Domain Admin accounts (if you tell it to), via modification of the users' LogonWorkstations attribute. This is explained further below. It will also apply a secure Group Policy Object (GPO) to your Domain Controllers OU - again if you tell it to. If you want to see the settings in the GPO, you can download it [here](https://github.com/curi0usJack/activedirectory/raw/master/DomainLockDown/GPOs/DomainLockDown-2008DCs.zip "Download GPO") and import it into a test GPO object in your domain. Note that as of today the GPO has been tested on a 2008 DC. It should work with 2012 (in theory), but has not yet been tested. FYI.


When the script is run with a remediate flag is run, it creates a GPO called "DomainLockDown-DCs" and will link it to your Domain Controllers OU. This GPO is based off the [Microsoft Security Compliance Manager](http://www.microsoft.com/en-us/download/details.aspx?id=16776 "Download Microsoft Security Compliance Manager"), and also includes disabled Credential Caching.

![Image 1](http://i.imgur.com/BLxA7VK.jpg "Image 1")


Additionally, it will also create a [fine grained password policy](http://technet.microsoft.com/en-us/library/cc770394(v=ws.10).aspx "AD DS: Fine-Grained Password Policies") object and apply it to your Domain Admins:

![Image 2](http://i.imgur.com/9qYHuNf.png "Image 2")


The password policy object is configured with the following parameters:

Parameter | Value
| --- | --- |
| Minimum Password Length: | 12 characters |
| Maximum Password Age:    | 30 Days |
| Complexity Enabled:      | Yes |
| Lockout Attempts:        | 5 Bad Attempts in 24 Hours. |
| Lockout Duration:        | Forever |
| Password Remembered:     | 10 |
| Minimum Password Age:    | 3 Days |
 

###PRE-REQUISITES

#####There are several prereqs:

+ It must be running on a Domain Controller.
+ It must run as a Domain Admin.
+ The ActiveDirectory and GroupPolicy cmdlets must be available (install [RSAT](http://www.microsoft.com/en-us/download/details.aspx?id=7887 "http://www.microsoft.com/en-us/download/details.aspx?id=7887")).
+ It must be able to write files to your $env:TEMP directory. Normally this is C:\Users\USERID\AppData\Local\Temp. 

(Note that a log is written for every DLD run and is saved to the current working directory.)

#####There are four switches in DLD:

__[-evaluate]__  The evaluate switch is a read-only check against your Domain. No changes are performed. Evaluate does a few things:
+ It evaluates the password policy that is in place for your DAs.
+ It looks at the number of DA accounts and gives feedback. This was obviously a judgement call on my part, but suffice to say I believe strongly that the less DA accounts there are, the better.
+ It checks to see where the DA accounts can login to and gives feedback if it's anywhere but DCs (read Part 1 to understand why).
+ It looks to see if anyone other than DAs and Enterprise Admins can login to DCs (via the Builtin\Administrators group). 
+ It checks if null sessions are enabled on your Domain and will list any currently connected null sessions.
+ It checks to see if credential caching is enabled on your Domain.


__[-remediate]__  The remediate switch will make changes to your domain, however you are prompted for each change:
+ It will add the password policy described above.
+ It unzips the DLD gpo & links it to the Domain Controllers OU. Note that creating the GPO and linking it are separate prompts. 
+ It will restrict your DAs to only be able to login to DCs. You are prompted for changes to each DA.

__[-undo]__  The undo switch will delete the password policy and the DomainLockDown GPO. It will not modify any changes to your DA accounts.

__[-deathblossum]__  The deathblossum switch does everything remediate does, except that it will only ask you once for confirmation (type in YES, case-sensitive), then it goes to work. Use with caution! You've been warned.
