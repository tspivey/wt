# WT
## Introduction
Wt is a web-based terminal for blind users.
The output is delivered via web speech, and there's no on-screen terminal display.
A modified tdsr is used to read the screen, running through Pyodide.

## Installation
You need a few things to get this going.

Serve the files in `src` on a web server somewhere. For testing,
`python3 -m http.server` will work.

We need a TCP to executable bridge. For that, install `socat`, and run:
`TERM=xterm socat TCP-LISTEN:8110,fork,reuseaddr,bind=127.0.0.1 EXEC:/bin/bash,pty,stderr,setsid`
This will listen on localhost port 8110, and run bash.

Next we need a TCP to web socket proxy. Install `websocat` and run:
`websocat -E --binary --restrict-uri /wtsocket ws-l:0.0.0.0:8112 tcp:127.0.0.1:8110`

This tells websocat to listen on all interfaces on port 8112, bridging to 127.0.0.1:8010, to drop the connection if one of them reaches EOF,
serves in binary mode, and restricts the URI to `/wtsocket`.

Port 8112 is hard-coded in the source. To change it, edit `src/index.html`.

Once that is running, open your browser to wherever you're serving the files, and press Start.

You should hear 1, 2 and 3 as various parts load, then tdsr should come up, say connected as it connects to the web socket
(which by default is on the same host as the server), and you should be dumped into bash.

## Usage notes
Use tdsr as normal. If you're on an iPhone, option will act as Alt.
The rate depends on the synth you're using. tdsr's rate is web speech rate * 100.
To change the voice, press alt c, shift v, and choose a number from 0 to however many voices you have. By default, the voices are restricted to English.

## Limitations
You need a keyboard. The touch screen phone keyboard won't work, it's not flexible enough on the web.
I can't add a row of keys on top to give Alt, Ctrl, etc.

some keys will always pass through to the browser. On PC, Control-w, t and n.
On iOS, Option+u will start a composition sequence, but pressing it twice reads the previous line.

## Security
The example above has bash listening on a TCP port and forwarded to a web socket.
Anyone with local shell access can gain access to the bash listening on localhost, and anyone who knows the URI prefix and has access to the web socket can gain access to bash through it.
To secure this, you can:
* Run over a VPN like Tailscale.
* Run something other than bash, maybe login, or a script that asks for a password.
* Change the URI prefix in the source code.

## Todo
* Try to handle compose events so Option+u will work
* Save config in local storage
* Improve voice selection, that can go in the GUI
