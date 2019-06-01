+++
title = "Notmuch in Emacs with mbsync and msmtp"
bookmarks = ["emacs"]
description = "Even I can do it."
date = "2019-05-31"
type = "essay"
+++
### Emails, Notmuch, and I
Emails are always distractions for me. Before using notmuch with Emacs, I don't use any email clients on my laptop, so I'd have to log into the office 365's web interface to read and reply emails, which is absolutely a pain in the neck. Yet I chose this workflow because it pretty much made checking email the least favorable thing whenever I am working, which was exactly what I needed.

Yet as I go through the transition to a senior phd student, there is simply no way to avoid emails any longer. I have to follow emails from advisors regarding various instructions on drafts, meetings, and numerous errands. There are also more emails from organizations as I start investing more time working in the professional societies. I need to not only constantly referring back to the emails, but also be able to archive them in a more efficient way (instead of struggling with tens of archive folders and which email goes to which folder).

I've been looking for a solution for emails. Notmuch's philosophy is tagging and instant searching. You don't ever need to worry how to set up your hierarchical structure of your archives and how to assign emails to the best child-folder. In notmuch, you do initial tag upon emails' arrival. You can then assign each email with multiple tags (while reading and replying) and notmuch will automatically generate different groups based on the tags. The "tag groups" resemble "archive folders," only in a much more flexible and dynamic way.

What's more? Boy, notmuch searching is blazing fast. I don't ever feel I need a static folder to hold emails: I just search for the keyword whenever I need any pieces of information.

All these being said, I am a total newbie in Emacs. Wait. I am a newbie in all sorts of code-related stuff. Yet as always, I am allured to any ideas that can improve my workflow no matter how many troubles are ahead of them. I ran into plenty of difficulties when setting notmuch because of my limited knowledge on programming. A lot of those difficulties are so basic that I couldn't find any solutions on the internet. This blog entry is for people like me who enjoy tweaking workflows but are not necessary familiar with programming. So here we go.

Basically, We need three things to perform three jobs:

+ Fetching emails: mbsync
+ Reading emails: notmuch
+ Replying emails: msmtp

Emacs' native message mode is also capable of sending emails but I find it sometimes buggy, so I end up using msmtp, which is surprisingly easy to configure. 
<br/>
<br/>
### Mbsync
#### install 
```bash 
brew install isync
```

#### configuration with office 365 account
```bash 
cd ~
touch .mbsyncrc
open .mbsyncrc
```

Edit your account information based on the following template, and paste it into .mbsyncrc

```bash
IMAPAccount outlook
	Host outlook.office365.com
	Port 993
	User YOURACCOUNT
	Pass YOURPASSWORD
	SSLType IMAPS
	AuthMechs LOGIN
	CertificateFile /usr/local/etc/openssl/cert.pem

IMAPStore outlook-remote
	Account outlook

MaildirStore outlook-local
	Path ~/.mail/default/
	Inbox ~/.mail/default/Inbox/
	SubFolders Verbatim

Channel outlook
	Master :outlook-remote:
	Slave :outlook-local:
# Include everything
	Patterns *
	Create Both
	Sync All
	Expunge Both
	SyncState *
```

As I am on macOS, the certification used is from the openSSL so the path is different from some other configuration files on the internet. You can easily instill openSSL via 
```bash
brew install openssl
```  


**This one is important.** An issue I ran into when I first set up the file was getting error message as 
```bash
unknown/misplaced keyword 'AuthMechs' 'SSLType'
``` 
To solve this problem, please refer to the configuration section on the latest official website [mbsync](http://isync.sourceforge.net/mbsync.html#CONFIGURATION), and try to:
  
+ reorder variables so that they follow the order listed on the websites, say Host needs to go before the Port variable, or AuthMechs has to go before the SSLType;
+ pay attention to your line spacing. On the website, it says "Sections are started by a section keyword and are terminated by an empty line or end of file," which means only one empty line is allowed between each section. Make sure, for example, that you don't have two empty lines between the section "IMAPAccount" and the section "IMAPstore"
+ If still having troubles, I recommend downloading existing profiles shared by others from the Github and ONLY replacing your account information. Try not to change any formatting of the file.
  
The variable `Subfolders Verbatim` is optional but necessary if your email account have many subfolders. Mbsync won't start fetching emails in the subfolders until you set the subfolder structure. You can find all three options on the configuration page of [mbsync](http://isync.sourceforge.net/mbsync.html#CONFIGURATION).

### run
```bash 
mbsync -a
```
All the emails will be downloaded to the specified path as maildir files.
<br/>
<br/>
## Notmuch
### install
```bash 
brew install notmuch
```

Upon finishing, terminal will display where notmuch puts the emcas files. In my case (yours may be different), it is 
```bash
/usr/local/share/emacs/site-lisp/notmuch
``` 
Next we need to link this folder to your ~/.emacs.d directory:

```bash
ln -s  /usr/local/share/emacs/site-lisp/notmuch/ ~/.emacs.d/
```

### configuration
Notmuch's website has detailed configuration guide. See [getting-started](https://notmuchmail.org/getting-started/).

Note that setting path during `notmuch` step is a little tricky. Assuming your mail directory is `~/Mails/` then please run `cd ~` before the command `notmuch` and input `Mails/` as the path. If the directory is `~/Desktop/Mails/` then be sure to `cd ~/Desktop` prior to the `notmuch` command. 
<br/>
<br/>
## Msmtp
### install
```bash
brew install msmtp
```

### configuration 
```bash 
cd ~
touch .msmtprc
open .msmtprc
```
And then simply edit and paste the following into the file:
```bash 
# outlook
account outlook
host smtp.office365.com
port 587
protocol smtp
auth on
from YOURACCOUNT
user YOURACCOUNT
password YOURPASSWORD
tls on
tls_nocertcheck
logfile ~/.msmtp.log
account default : outlook 
```

### test 
To test if msmtp is working, create a `txt` file, say `test.txt` :
```bash
To: example@gmail.com
From: YOURACCOUNT
Subject: a testing email
Hi there :)
```
And send it with the command:

```bash 
cat test.txt | msmtp -a default example@gmail.com
```

I recommend using the above command to send test emails as the well-documented `echo` function sometimes sends empty emails without subjects and contents.
<br/>
<br/>
## Finally, Emacs
### notmuch
In your init.el file, add the following codes
```html
;;notmuch setup
(add-to-list 'exec-path "/usr/local/bin/")
(setq notmuch-binary "/usr/local/bin/notmuch") 
(add-to-list 'load-path (expand-file-name "~/.emacs.d/notmuch"))
(autoload 'notmuch "notmuch" "notmuch mail" t)
(require 'notmuch)
```
We need to first set the path as we installed Notmuch via homebrew and sometimes emacs cannot find the directory. The `'load-path`  variable specifies the folder in the `~/.emacs.d/`  that is linked from the `/usr/local/share/emacs/site-lisp/notmuch` 

Also refer to the official website for [notmuch-emacs](https://notmuchmail.org/notmuch-emacs/) interface.

### msmtp
Msmtp can be configured in notmuch's configure file altogether.
```bash 
cd ~/.emacs.d
touch notmuch-config.el
```
In this file, add the following settings:
```html
(setq user-full-name "YOURNAME")
(setq user-mail-adress "YOURACCOUNT")
(setq mail-user-agent 'message-user-agent)
(setq mail-specify-envelope-from t)
(setq sendmail-program "/usr/local/bin/msmtp"
	  mail-specify-envelope-from t
	  mail-envelope-from 'header
	  message-sendmail-envelope-from 'header)
```
<br/>
<br/>
## Good to go
By far the notmuch-emacs system should work properly. Type 
```
M-x notmuch
```
and notmuch's hello face will be waiting 📩📬📧📨 :)
