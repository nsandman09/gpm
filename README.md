#GPM: Generic Package Manager

This is a lightweight package manager designed to be easy to use and easy to make packages for. It is inspired by Homebrew, but is different in many regards:

1. Packages are JSON files with one URL and bash commands, not Ruby scripts with methods.
2. Packages are stored on a [server](http://github.com/gpm-packages), and you do **not** have to fork it to add custom packages.
3. GPM is written in Python, not Ruby, and is significantly smaller.
4. GPM modifies your $PATH variable instead of symlinking all installs.

##Installation
Installation is easy. Just run:

```bash
curl http://pastebin.com/raw.php?i=njyLkDFY | bash
```

Then, to add GPM to your path, restart or run:

```bash
. ~/.profile
```

###So why is it "generic"?
I dunno. The name "GPM" just sounds cool to me. I guess you could say it installs packages not for specific languages/environments (unlike, npm, pip, etc.). 

###Huh. So how do I make my own package?
Easy! A simple package called "testing" (which, when run, echoes the word "testing" back.) would look like this:

```javascript
{
	"url": "http://links.bargonaut.com/gpm/testing",
	"commands": [
		"chmod +x {FILENAME}"
	]
}
```
Looks simple enough. Let's break this down.

GPM will first download the file in the URL. Then, it will run the commands in the array of the same name, giving you access to two variables:

* {FILENAME}: This is the path to the file downloaded from the URL.
* {GPMDIR}: This is the path to the GPM installation directory. This is usually ~/.gpm.

Let's write a slightly more complex one. This downloads and installs node.js:

```javascript
{
	"url": "https://semver.io/node/stable",
	"commands": [
		"echo \"Downloading latest node.js source...\"",
		"curl --progress-bar http://nodejs.org/dist/v`cat {FILENAME}`/node-v`cat {FILENAME}`.tar.gz > {GPMDIR}/tmp/node.tgz ",
		"echo \"Extracting...\"",
		"tar -xf {GPMDIR}/tmp/node.tgz -C {GPMDIR}/tmp",
		"echo \"Done. Building (be patient!)...\"",
		"{GPMDIR}/tmp/node-v`cat {FILENAME}`/configure --prefix={GPMDIR}/installed",
		"make -C {GPMDIR}/tmp/node-v`cat {FILENAME}`",
		"make -C install {GPMDIR}/tmp/node-v`cat {FILENAME}`",
		"rm {FILENAME}"
	]
}
``` 
It looks like a lot, but don't be daunted. Remove all the "echo" commands and it's easier to make sense of:

```javascript
{
	"url": "https://semver.io/node/stable",
	"commands": [
		"curl --progress-bar http://nodejs.org/dist/v`cat {FILENAME}`/node-v`cat {FILENAME}`.tar.gz > {GPMDIR}/tmp/node.tgz ",
		"tar -xf {GPMDIR}/tmp/node.tgz -C {GPMDIR}/tmp",
		"{GPMDIR}/tmp/node-v`cat {FILENAME}`/configure --prefix={GPMDIR}/installed",
		"make -C {GPMDIR}/tmp/node-v`cat {FILENAME}`",
		"make -C install {GPMDIR}/tmp/node-v`cat {FILENAME}`",
		"rm {FILENAME}"
	]
}
```
Okay, that's easier to read. Now let's look at this. It downloads the latest node version from http://www.semver.io. This is in the "url" field. So everywhere it says \`cat {FILENAME}`, it's just putting the latest node.js version. So, assuming that's **0.12.2**, this will show up as:

```javascript
{
	"url": "https://semver.io/node/stable",
	"commands": [
		"curl --progress-bar http://nodejs.org/dist/v0.12.2/node-v0.12.2.tar.gz > {GPMDIR}/tmp/node.tgz ",
		"tar -xf {GPMDIR}/tmp/node.tgz -C {GPMDIR}/tmp",
		"{GPMDIR}/tmp/node-v0.12.2/configure --prefix={GPMDIR}/installed",
		"make -C {GPMDIR}/tmp/node-v0.12.2",
		"make -C install {GPMDIR}/tmp/node-v0.12.2",
		"rm {FILENAME}"
	]
}
```
So it is now more obvious. All this script does is download the source, extract it, and run "./configure", "make", and "make install" with a few custom options.