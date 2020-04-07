+++
title = "Setting up Org-protocal with Firefox on macOS"
bookmarks = ["emacs"]
description = "Even I can do it."
date = "2020-04-05"
type = "essay"
+++
### TL;DR (Organizing Annotated Bibliography in Org-mod)
I've had trouble establishing an effective work flow to organize annotated bibliography with Emacs. Previously I've been using [Org-brain](https://github.com/Kungsgeten/org-brain), a sophisticated wiki-style tool to visualize concepts and their relationships. Here is a screenshot of the structure I set up. 

![Org-brain overview](https://apfbvvpren.cloudimg.io/v7/raw.githubusercontent.com/wpix/solid-pipix/master/articles/org-brain-overview.png?width/cdn/n/n)

Though one can interlink all concepts, it is hard to accurately locate a topic and related ones if one set up the hierarchy more than three levels, which is common in scientific research. I've tried logging multiple annotated bibliography under one parent headline or logging each bibliography as a standalone entry, but nothing quite work the way I imagined. I'd recommend Org-brain to users who mainly want to map relationships between concepts or ideas, with less focuses on details or descriptions of themselves. 

Another option is to import bibliography into [Org-ref](https://github.com/jkitchin/org-ref) and annotate them in a org file. It's easy to set up and works smoothly but having all the bibliography stored in one single file makes things mixed up, especially when you switching to a different project or working simultaneously on multiple ones. (I highly recommend Org-ref for organizing bibliography and org-mode citation, a must-have for academic usage.) 

I tend to download every related paper during a search session, only leaving most of which unread. It occurred to me that "downloading" those references makes it okay to "postpone" reading them, like forever. In fact, sorting and summarizing needed information can be well-achieved when skimming the papers. Therefore, I ended up picking up the most common solution, writing down annotated bibliography in one org file for each project. In this case, being able to conveniently capture selected sources or text outside Emacs became the priority, because, let's be honest, web-browsing in Emacs is pretty insane. 
<br/>
<br/>
### Configuration Overview
The [org-protocol-capture-html](https://github.com/alphapapa/org-protocol-capture-html) package seems like exactly what I need. Converting _html_ pages or selected _html_ text in Chrome or Firefox browsers to _org-mode_ format text via Pandoc, Org-protocal, and Org-capture. 

This blog will guide you through setting:

+ Emacs external handler with Org-protocal

+ Org-capture template

+ Firefox configuration

+ Pandoc (if not already installed)

With few guides available for setting this up for macOS, I record here the configurations that worked for me and hopefully it will be any help if you happen to find this blog. Below is my system information, FYI.

> System Version: macOS 10.14.4 (18E226) 
>
> Kernel Version | Darwin 18.5.0 
<br/>

### Installing Org-protocal on macOS
Org-protocal is the agent between external applications and Emacs, meaning you can trigger Emacs actions outside of it. The [official page](https://orgmode.org/worg/org-contrib/org-protocol.html) is a bit outdated, though. 

The [guide](https://blog.aaronbieber.com/2016/11/24/org-capture-from-anywhere-on-your-mac.html) recommended by Org-protocal-capture-html no longer works for me. I found the [tutorial of org-capture extension](https://github.com/sprig/org-capture-extension#set-up-org-protocol) useful.

Here we go.
```bash 
cd ~/.local/share/applications
``` 
and create a desktop file **org-protocol.desktop**. Open this file with TextEdit and paste the following text into the file, then save.

```
[Desktop Entry]
Name=org-protocol
Exec=emacsclient %u
Type=Application
Terminal=false
Categories=System;
MimeType=x-scheme-handler/org-protocol;
```

If you haven't already installed the desktop utensils, run 
``` bash
brew install desktop-file-utils
```
Then update the application folder with 

``` bash
update-desktop-database ~/.local/share/applications/
```
If succeeded, a file named "mimeinfo.cache" will be created in this folder. 

#### Preparing the emacsclient application
You can continue following the [guide here](https://github.com/sprig/org-capture-extension#under-osx) creating your own emacsclient application or you can download the prepared [dmg package](https://github.com/sprig/org-capture-extension/raw/master/EmacsClient.app.zip) directly. I recommend the latter.

<br/>
### Emacs configuration 
In your init file or equivalent, adding

``` diff
(server-start)
(require 'org-protocol)
```
and,

``` diff
(setq org-directory "~/org-notes")
(setq org-default-notes-file (concat org-directory "/notes.org"))
(define-key global-map "\C-cc" 'org-capture)

(defun transform-square-brackets-to-round-ones(string-to-transform)
  "Transforms [ into ( and ] into ), other chars left unchanged."
  (concat 
  (mapcar #'(lambda (c) (if (equal c ?[) ?\( (if (equal c ?]) ?\) c))) 
  string-to-transform)))

(setq org-capture-templates
      '(("w" "Web site" entry
	  (file+olp "~/org-notes/notes.org" "Web")
	  "* %c :website:\n%U %?%:initial")))

(setq org-protocol-default-template-key "w")
```

In terminal, input 
``` bash 
emacsclient -n "org-protocol:///capture?url=https://www.wikipedia.org/"
``` 
and a Org-capture buffer should appear in the current or a new Emacs window. Replace any url you want to test after **url=**.

If the capture buffer doesn't pop up or the capture template is wrong, make sure to check what goes wrong in earlier setting. 

### Firefox integration
This is achieved by a javascript bookmarklet. Before creating the bookmarklet, type **about:config** in Firefox url address space. Upon entering it, you will likely see a blank page with a search bar on top. In that search area, type **network.protocol-handler.expose.org-protocol** and choose the **boolean** option, then setting the value to **true**.

To sure to do this step if you don't want all your current tabs go missing if any issues occurs when testing the bookmarklet. 

Right-click menu bar and select **add a new bookmark ...** and pasting the following javescript into the **Location** region:

``` javascript
javascript:location.href = 'org-protocol://capture-eww-readable?template=w&url=' + encodeURIComponent(location.href) + '&title=' + encodeURIComponent(document.title || "[untitled page]");
```

#### Pandoc Installation*
It is unlikely, but in case the [Pandoc](https://pandoc.org/installing.html#chrome-os) is not installed, run 
```bash 
 brew install pandoc
```

### Final testing
Open any webpage and hit the bookmarklet, a organized webpage in org-mode should appear in Emacs. 

Here is a screenshot capturing this blog entry.

![org-capture-example](https://apfbvvpren.cloudimg.io/v7/raw.githubusercontent.com/wpix/solid-pipix/master/articles/org-capture-overview.png?width/cdn/n/n)

Hopefully I will be capturing a ton literature considering the time I throw into configuring this feature.

Enjoy~
