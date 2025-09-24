---
3D"description": 3D\"Random
3D"generator": 3D\"Jekyll
3D"twitter:card": 3D\"summary_large_image\"
3D"viewport": 3D\"width=3Ddevice-width,
lang: 3D\"en-US\"
title: Set Network to use DHCP \| lkiesow =E2=8B=84 weblog
---

From: Snapshot-Content-Location:
https://weblog.lkiesow.de/20220223-proxmox-test-machine-self-servic/proxmox-server-dhcp.html
Subject:
=?utf-8?Q?Set=20Network=20to=20use=20DHCP=20\|=20lkiesow=20=E2=8B=84=20web?=
=?utf-8?Q?log?= Date: Wed, 24 Sep 2025 11:58:40 +0300 MIME-Version: 1.0
Content-Type: multipart/related; type=\"text/html\";
boundary=\"\-\-\--MultipartBoundary\--cq6K1Gf6bh4FhITIQvwBOV2ZLTktzovsSTb1dYK5Lk\-\-\--\"
\-\-\-\-\--MultipartBoundary\--cq6K1Gf6bh4FhITIQvwBOV2ZLTktzovsSTb1dYK5Lk\-\-\--
Content-Type: text/html Content-ID: Content-Transfer-Encoding:
quoted-printable Content-Location:
https://weblog.lkiesow.de/20220223-proxmox-test-machine-self-servic/proxmox-server-dhcp.html

::: {#3D\"header\"}
[View On
GitHub](3D%22https://github.com/lkiesow/weblog.l=){kiesow.de\"=""}

=20
:::

::: 3D"wrapper"
::: section
[](3D%22https://weblog.lkiesow.de/%22){#3D\"title\"}

# lkiesow =E2=8B=84 weblog

Random notes from a security-aware software engineer, open-sou= rce
advocate and occasional lecturer.

------------------------------------------------------------------------

# Set Network to use DHCP {#3D\"set-network-to-use-dhcp\"}

> **=E2=9A=A0 Warning:** Remember that you are modifying yo= ur network
> configuration. Make sure to have a backup plan in case you lock
> yourself out.

Having changing IP addresses can cause problems for Proxmox. That is why
the default behavior is to configure static addresses. Please evaluate
your environment before following this guide.

## Why I went with DHCP {#3D\"why-i-went-with-dhcp\"}

I wanted to bring some Proxmox servers to an enterprise environment.
This means that if I want to get a device deployed, I have to:

-   Talk to one of our network managers
-   Get my device (MAC address) added to our IP address management
    (IPAM)= solution
-   Get an IP address based on the network segment I want to be in
-   Configure the address or enable DHCP

This means that even with DHCP, I have a static IP address guaranteed
un= less I specifically request a change.

Still, my first deployment used static IP addresses. That worked great
until I requested the server to be moved from the data ce= nter internal
network to the external network so that our developers can ac= cess the
machine from home. That is when I get a net IP in a different network
segment assigned. That is also, when my server had an invalid network
configuration, and I wa= s unable to access it.

Not great when you work from home and access to your data center is
rest= ricted to internal personal only. Luckily, I still had access
through the server=E2=80=99s (horrible) remote = management console and
was able to resolve the issue. Nevertheless, that drove me to using
static IPs assigned via DHCP.

By now, I almost always use DHCP for Proxmox, using my network
infrastru= cture to assign static IP addresses. I do that even at home,
where accessing a device directly is relatively eas= y.

## Network Configuration {#3D\"network-configuration\"}

To enable DHCP, on your server, edit
`/etc/network/interfaces`{.3D\"language-plaintext
h="ighlighter-rouge\""}. You should see a configuration like this
(interface names may varry):

::: {.3D\"language-plaintext highlighter-rouge\"=""}
::: {.3D\"highlight= \"=""}
``` 3D"highlight"
iface vmbr0 inet static
        address 192.168.1.157/24
        gateway 192.168.1.1
        bridge-ports enp5s0
        bridge-stp off
        bridge-fd 0
```
:::
:::

Modify this block and turn it into a DHCP configuration:

::: {.3D\"language-plaintext highlighter-rouge\"=""}
::: {.3D\"highlight= \"=""}
``` 3D"highlight"
iface vmbr0 inet dhcp
        bridge-ports enp5s0
        bridge-stp off
        bridge-fd 0
```
:::
:::

## Configure Hostname {#3D\"configure-hostname\"}

You should also make sure your hostname is properly configured. You can
set the hostname using the tool `hostnamectl`{.3D\"language-plaintext
h="ighlighter-rouge\""}:

``` 3D"language-term"
=E2=9D=AF hostnamectl set-hostname proxmox.home.lkiesow.io
```

To verify it is set correctly, use:

``` 3D"language-term"
=E2=9D=AF hostname
proxmox.home.lkiesow.io
```

Finally, make sure, the hostname is entered correctly in
`/etc/hosts`{.3D\"= language-plaintext="" highlighter-rouge\"=""}. The
file should look somewhat like this:

::: {.3D\"language-plaintext highlighter-rouge\"=""}
::: {.3D\"highlight= \"=""}
``` 3D"highlight"
127.0.0.1      localhost.localdomain local=
host
192.168.1.157  proxmox.home.lkiesow.io proxmox
```
:::
:::

## Dynamic Host Configuration {#3D\"dynamic-host-configuration\"}

On a Proxmox server, when updating the IP address,
`/etc/hosts`{.3D\"langua= ge-plaintext="" highlighter-rouge\"=""} must
be updated as well. That is why just enabling DHCP can cause problems.

If you can use your infrastructure to ensure IPs do not change, that=E2=
=80=99s great. If not, you can use dhcpclient hooks to automatically
update this file. To do that, create a new file
`/etc/dhcp/dhclient-exit-hooks.d/update-etc-hosts`{.3D\"language-plaintext
highlighter="-rouge\""} with conten= t like this:

::: {.3D\"language-sh highlighter-rouge\"=""}
::: 3D"highlight"
``` {.3D\"highlight\" ==""}
if ([<=
/span> $reason =3D "BOUND" ] |=
| [ $reason =3D "RENEW" ])
then
  sed -i "s/^.*\spro=
xmox.home.lkiesow.io\s.=
*$/${new_ip_address} =
proxmox.home.lkiesow.io proxmox/" /etc/hosts
fi
```
:::
:::

I don=E2=80=99t usually use this, but it may be helpful in some cases.

------------------------------------------------------------------------

[=E2=97=82   Back to: Part 1 =E2=80=93 = Basic Set-up of
Proxmox](3D%22https://weblog.lkiesow.de/20220223-proxmox-test-machine-self-=){servic=""
part-1-basic-setup.html\"=""}
:::

=20 [=E2=97=80 Back](3D%22https://weblog.lkiesow.de/%22) =20
:::

=20

::: {#3D\"syno-notification-is-installed\"}
:::

\-\-\-\-\--MultipartBoundary\--cq6K1Gf6bh4FhITIQvwBOV2ZLTktzovsSTb1dYK5Lk\-\-\--
Content-Type: image/gif Content-Transfer-Encoding: base64
Content-Location: https://weblog.lkiesow.de/assets/images/nav-bg.gif
R0lGODlhNAA0AIAAACoqKjU1NSH/C1hNUCBEYXRhWE1QPD94cGFja2V0IGJlZ2luPSLvu78iIGlk
PSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0YSB4bWxuczp4PSJhZG9iZTpu
czptZXRhLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS4wLWMwNjEgNjQuMTQwOTQ5LCAyMDEw
LzEyLzA3LTEwOjU3OjAxICAgICAgICAiPiA8cmRmOlJERiB4bWxuczpyZGY9Imh0dHA6Ly93d3cu
dzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRmOkRlc2NyaXB0aW9uIHJkZjph
Ym91dD0iIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20veGFwLzEuMC8iIHhtbG5zOnht
cE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4bWxuczpzdFJlZj0iaHR0cDov
L25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVmIyIgeG1wOkNyZWF0b3JUb29s
PSJBZG9iZSBQaG90b3Nob3AgQ1M1LjEgTWFjaW50b3NoIiB4bXBNTTpJbnN0YW5jZUlEPSJ4bXAu
aWlkOjQ1MDc1NkIzNjU3NTExRTFBQjJFOTM1RTIxNzM5QUFEIiB4bXBNTTpEb2N1bWVudElEPSJ4
bXAuZGlkOjQ1MDc1NkI0NjU3NTExRTFBQjJFOTM1RTIxNzM5QUFEIj4gPHhtcE1NOkRlcml2ZWRG
cm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6NDUwNzU2QjE2NTc1MTFFMUFCMkU5MzVFMjE3
MzlBQUQiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6NDUwNzU2QjI2NTc1MTFFMUFCMkU5MzVF
MjE3MzlBQUQiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJERj4gPC94OnhtcG1ldGE+IDw/
eHBhY2tldCBlbmQ9InIiPz4B//79/Pv6+fj39vX08/Lx8O/u7ezr6uno5+bl5OPi4eDf3t3c29rZ
2NfW1dTT0tHQz87NzMvKycjHxsXEw8LBwL++vby7urm4t7a1tLOysbCvrq2sq6qpqKempaSjoqGg
n56dnJuamZiXlpWUk5KRkI+OjYyLiomIh4aFhIOCgYB/fn18e3p5eHd2dXRzcnFwb25tbGtqaWhn
ZmVkY2JhYF9eXVxbWllYV1ZVVFNSUVBPTk1MS0pJSEdGRURDQkFAPz49PDs6OTg3NjU0MzIxMC8u
LSwrKikoJyYlJCMiISAfHh0cGxoZGBcWFRQTEhEQDw4NDAsKCQgHBgUEAwIBAAAh+QQAAAAAACwA
AAAANAA0AAACj0RueJG8DZ9U1E2LY9X58r55Ykhi4MmlY6aWLbrC7vp+NY3PuczXt96L/YJA226I
XBCXyGTxqHQyd9On9Sr1QbHU7JZXDb/E35SXWyKjT+p2w11+nOHx9XiOj9bdebveDxjYJ7gHMDhH
15aIeFjWuLa4Fwn5yFQpNRmWiXl51Hm1+RUK+tlTOjS6lIp6SlMAADs=
\-\-\-\-\--MultipartBoundary\--cq6K1Gf6bh4FhITIQvwBOV2ZLTktzovsSTb1dYK5Lk\-\-\--
Content-Type: image/gif Content-Transfer-Encoding: base64
Content-Location: https://weblog.lkiesow.de/assets/images/hr.gif
R0lGODlhBQADAKIAACQjJCQkJCUkJSEhISUlJUNDQwAAAAAAACH/C1hNUCBEYXRhWE1QPD94cGFj
a2V0IGJlZ2luPSLvu78iIGlkPSJXNU0wTXBDZWhpSHpyZVN6TlRjemtjOWQiPz4gPHg6eG1wbWV0
YSB4bWxuczp4PSJhZG9iZTpuczptZXRhLyIgeDp4bXB0az0iQWRvYmUgWE1QIENvcmUgNS4wLWMw
NjEgNjQuMTQwOTQ5LCAyMDEwLzEyLzA3LTEwOjU3OjAxICAgICAgICAiPiA8cmRmOlJERiB4bWxu
czpyZGY9Imh0dHA6Ly93d3cudzMub3JnLzE5OTkvMDIvMjItcmRmLXN5bnRheC1ucyMiPiA8cmRm
OkRlc2NyaXB0aW9uIHJkZjphYm91dD0iIiB4bWxuczp4bXA9Imh0dHA6Ly9ucy5hZG9iZS5jb20v
eGFwLzEuMC8iIHhtbG5zOnhtcE1NPSJodHRwOi8vbnMuYWRvYmUuY29tL3hhcC8xLjAvbW0vIiB4
bWxuczpzdFJlZj0iaHR0cDovL25zLmFkb2JlLmNvbS94YXAvMS4wL3NUeXBlL1Jlc291cmNlUmVm
IyIgeG1wOkNyZWF0b3JUb29sPSJBZG9iZSBQaG90b3Nob3AgQ1M1LjEgTWFjaW50b3NoIiB4bXBN
TTpJbnN0YW5jZUlEPSJ4bXAuaWlkOjg0QzlEN0VBNjU1RTExRTFBQjJFOTM1RTIxNzM5QUFEIiB4
bXBNTTpEb2N1bWVudElEPSJ4bXAuZGlkOjg0QzlEN0VCNjU1RTExRTFBQjJFOTM1RTIxNzM5QUFE
Ij4gPHhtcE1NOkRlcml2ZWRGcm9tIHN0UmVmOmluc3RhbmNlSUQ9InhtcC5paWQ6ODRDOUQ3RTg2
NTVFMTFFMUFCMkU5MzVFMjE3MzlBQUQiIHN0UmVmOmRvY3VtZW50SUQ9InhtcC5kaWQ6ODRDOUQ3
RTk2NTVFMTFFMUFCMkU5MzVFMjE3MzlBQUQiLz4gPC9yZGY6RGVzY3JpcHRpb24+IDwvcmRmOlJE
Rj4gPC94OnhtcG1ldGE+IDw/eHBhY2tldCBlbmQ9InIiPz4B//79/Pv6+fj39vX08/Lx8O/u7ezr
6uno5+bl5OPi4eDf3t3c29rZ2NfW1dTT0tHQz87NzMvKycjHxsXEw8LBwL++vby7urm4t7a1tLOy
sbCvrq2sq6qpqKempaSjoqGgn56dnJuamZiXlpWUk5KRkI+OjYyLiomIh4aFhIOCgYB/fn18e3p5
eHd2dXRzcnFwb25tbGtqaWhnZmVkY2JhYF9eXVxbWllYV1ZVVFNSUVBPTk1MS0pJSEdGRURDQkFA
Pz49PDs6OTg3NjU0MzIxMC8uLSwrKikoJyYlJCMiISAfHh0cGxoZGBcWFRQTEhEQDw4NDAsKCQgH
BgUEAwIBAAAh+QQAAAAAACwAAAAABQADAAADCFhaA0QiLJYAADs=
\-\-\-\-\--MultipartBoundary\--cq6K1Gf6bh4FhITIQvwBOV2ZLTktzovsSTb1dYK5Lk\-\-\--
Content-Type: image/png Content-Transfer-Encoding: base64
Content-Location: https://weblog.lkiesow.de/assets/images/bullet.png
iVBORw0KGgoAAAANSUhEUgAAAAoAAAAJCAYAAAALpr0TAAAAUklEQVQYV2N48eJFGggzIAFsYiDB
u0D8H4iNoXxBdDGYQmMgfgfFSrjEkK0CmXAGZCIuMeJMRXOTCxZ3usBM2w0VgPsSmxhIcCYII/EF
0cVIAgBALbD3dOcVXAAAAABJRU5ErkJggg==
\-\-\-\-\--MultipartBoundary\--cq6K1Gf6bh4FhITIQvwBOV2ZLTktzovsSTb1dYK5Lk\-\-\--
Content-Type: text/css Content-Transfer-Encoding: quoted-printable
Content-Location:
https://weblog.lkiesow.de/assets/css/style.css?v=2122c0bfe3e95811a8263fda8e1b0f1e5d961a47
\@charset \"utf-8\"; article, aside, details, figcaption, figure,
footer, header, hgroup, nav, s= ection, summary { display: block; }
audio, canvas, video { display: inline-block; } audio:not(\[controls\])
{ display: none; } \[hidden\] { display: none; } html { font-size: 100%;
text-size-adjust: 100%; } html, button, input, select, textarea {
font-family: sans-serif; } body { margin: 0px; } a:focus { outline:
dotted thin; } a:hover, a:active { outline: 0px; } h1 { font-size: 2em;
margin: 0.67em 0px; } h2 { font-size: 1.5em; margin: 0.83em 0px; } h3 {
font-size: 1.17em; margin: 1em 0px; } h4 { font-size: 1em; margin:
1.33em 0px; } h5 { font-size: 0.83em; margin: 1.67em 0px; } h6 {
font-size: 0.75em; margin: 2.33em 0px; } abbr\[title\] { border-bottom:
1px dotted; } b, strong { font-weight: bold; } blockquote { margin: 1em
40px; } dfn { font-style: italic; } mark { background: rgb(255, 255, 0);
color: rgb(0, 0, 0); } p, pre { margin: 1em 0px; } pre, code, kbd, samp
{ font-family: monospace, serif; font-size: 1em; } q { quotes: none; }
q::before, q::after { content: none; } small { font-size: 75%; } sub,
sup { font-size: 75%; line-height: 0; position: relative; vertical-ali=
gn: baseline; } sup { top: -0.5em; } sub { bottom: -0.25em; } dl, menu,
ol, ul { margin: 1em 0px; } dd { margin: 0px 0px 0px 40px; } menu, ol,
ul { padding: 0px 0px 0px 40px; } nav ul, nav ol { list-style: none
none; } img { border: 0px; } svg:not(:root) { overflow: hidden; } figure
{ margin: 0px; } form { margin: 0px; } fieldset { border: 1px solid
rgb(192, 192, 192); margin: 0px 2px; padding: = 0.35em 0.625em 0.75em; }
legend { border: 0px; padding: 0px; white-space: normal; } button,
input, select, textarea { font-size: 100%; margin: 0px; vertical-al=
ign: baseline; } button, input { line-height: normal; } button,
input\[type=3D\"button\"\], input\[type=3D\"reset\"\],
input\[type=3D\"submit= \"\] { cursor: pointer; appearance: button; }
button\[disabled\], input\[disabled\] { cursor: default; }
input\[type=3D\"checkbox\"\], input\[type=3D\"radio\"\] { box-sizing:
border-box; p= adding: 0px; } input\[type=3D\"search\"\] { appearance:
textfield; box-sizing: content-box; }
input\[type=3D\"search\"\]::-webkit-search-decoration,
input\[type=3D\"search\"\]::= -webkit-search-cancel-button { appearance:
none; } textarea { overflow: auto; vertical-align: top; } table {
border-collapse: collapse; border-spacing: 0px; } \@font-face {
font-family: OpenSansLight; src: url(\"../fonts/OpenSans-Light-=
webfont.woff\") format(\"woff\"),
url(\"../fonts/OpenSans-Light-webfont.ttf\") f= ormat(\"truetype\");
font-weight: normal; font-style: normal; } \@font-face { font-family:
OpenSansLightItalic; src: url(\"../fonts/OpenSans-=
LightItalic-webfont.woff\") format(\"woff\"),
url(\"../fonts/OpenSans-LightItal= ic-webfont.ttf\")
format(\"truetype\"); font-weight: normal; font-style: norma= l; }
\@font-face { font-family: OpenSansRegular; src:
url(\"../fonts/OpenSans-Regu= lar-webfont.woff\") format(\"woff\"),
url(\"../fonts/OpenSans-Regular-webfont.t= tf\") format(\"truetype\");
font-weight: normal; font-style: normal; } \@font-face { font-family:
OpenSansItalic; src: url(\"../fonts/OpenSans-Itali= c-webfont.woff\")
format(\"woff\"), url(\"../fonts/OpenSans-Italic-webfont.ttf\"= )
format(\"truetype\"); font-weight: normal; font-style: normal; }
\@font-face { font-family: OpenSansSemibold; src:
url(\"../fonts/OpenSans-Sem= ibold-webfont.woff\") format(\"woff\"),
url(\"../fonts/OpenSans-Semibold-webfon= t.ttf\") format(\"truetype\");
font-weight: normal; font-style: normal; } \@font-face { font-family:
OpenSansSemiboldItalic; src: url(\"../fonts/OpenSa=
ns-SemiboldItalic-webfont.woff\") format(\"woff\"),
url(\"../fonts/OpenSans-Sem= iboldItalic-webfont.ttf\")
format(\"truetype\"); font-weight: normal; font-sty= le: normal; }
\@font-face { font-family: OpenSansBold; src:
url(\"../fonts/OpenSans-Bold-we= bfont.woff\") format(\"woff\"),
url(\"../fonts/OpenSans-Bold-webfont.ttf\") form= at(\"truetype\");
font-weight: normal; font-style: normal; } \@font-face { font-family:
OpenSansBoldItalic; src: url(\"../fonts/OpenSans-B=
oldItalic-webfont.woff\") format(\"woff\"),
url(\"../fonts/OpenSans-BoldItalic-= webfont.ttf\")
format(\"truetype\"); font-weight: normal; font-style: normal; = }
.highlight table td { padding: 5px; } .highlight table pre { margin:
0px; } .highlight, .highlight .w { color: rgb(208, 208, 208); }
.highlight .err { color: rgb(21, 21, 21); background-color: rgb(172, 65,
66= ); } .highlight .c, .highlight .cd, .highlight .cm, .highlight .c1,
.highlight .= cs { color: rgb(136, 136, 136); } .highlight .cp { color:
rgb(244, 191, 117); } .highlight .nt { color: rgb(244, 191, 117); }
.highlight .o, .highlight .ow { color: rgb(208, 208, 208); } .highlight
.p, .highlight .pi { color: rgb(208, 208, 208); } .highlight .gi {
color: rgb(144, 169, 89); } .highlight .gd { color: rgb(172, 65, 66); }
.highlight .gh { color: rgb(106, 159, 181); font-weight: bold; }
.highlight .k, .highlight .kn, .highlight .kp, .highlight .kr,
.highlight .= kv { color: rgb(170, 117, 159); } .highlight .kc { color:
rgb(210, 132, 69); } .highlight .kt { color: rgb(210, 132, 69); }
.highlight .kd { color: rgb(210, 132, 69); } .highlight .s, .highlight
.sb, .highlight .sc, .highlight .sd, .highlight .= s2, .highlight .sh,
.highlight .sx, .highlight .s1 { color: rgb(144, 169, 8= 9); }
.highlight .sr { color: rgb(117, 181, 170); } .highlight .si { color:
rgb(143, 85, 54); } .highlight .se { color: rgb(143, 85, 54); }
.highlight .nn { color: rgb(244, 191, 117); } .highlight .nc { color:
rgb(244, 191, 117); } .highlight .no { color: rgb(244, 191, 117); }
.highlight .na { color: rgb(106, 159, 181); } .highlight .m, .highlight
.mf, .highlight .mh, .highlight .mi, .highlight .= il, .highlight .mo,
.highlight .mb, .highlight .mx { color: rgb(144, 169, 8= 9); }
.highlight .ss { color: rgb(144, 169, 89); } body { padding: 0px 0px
20px; margin: 0px; font: 14px / 1.5 OpenSansRegular= , \"Helvetica
Neue\", Helvetica, Arial, sans-serif; color: rgb(240, 231, 213)= ;
background-image: linear-gradient(rgb(42, 42, 41), rgb(28, 28, 28));
back= ground-position: initial; background-size: initial;
background-repeat: init= ial; background-origin: initial;
background-clip: initial; background-color= : initial;
background-attachment: fixed !important; } h1, h2, h3, h4, h5, h6 {
color: rgb(232, 232, 232); margin: 0px 0px 10px; f= ont-family:
OpenSansRegular, \"Helvetica Neue\", Helvetica, Arial, sans-serif= ;
font-weight: normal; } p, ul, ol, table, pre, dl { margin: 0px 0px 20px;
} h1, h2, h3 { line-height: 1.1; } h1 { font-size: 28px; } h2 {
font-size: 24px; } h4, h5, h6 { color: rgb(232, 232, 232); } h3 {
font-size: 18px; line-height: 24px; font-weight: normal; color: rgb(18=
2, 182, 182); font-family: OpenSansRegular, \"Helvetica Neue\",
Helvetica, Ar= ial, sans-serif !important; } a { color: rgb(255, 204,
0); font-weight: 400; text-decoration: none; } a:hover { color: rgb(255,
235, 155); } a small { font-size: 11px; color: rgb(102, 102, 102);
margin-top: -0.6em; d= isplay: block; } ul { list-style-image:
url(\"../images/bullet.png\"); } strong { font-family: OpenSansBold,
\"Helvetica Neue\", Helvetica, Arial, san= s-serif !important;
font-weight: normal; } .wrapper { max-width: 650px; margin: 0px auto;
position: relative; padding:= 0px 20px; } section img { max-width: 100%;
} blockquote { border-left: 3px solid rgb(255, 204, 0); margin: 0px;
padding:= 0px 0px 0px 20px; font-style: italic; } code { font-family:
Monaco, \"Bitstream Vera Sans Mono\", \"Lucida Console\", T= erminal,
monospace; color: rgb(239, 239, 239); font-size: 13px; margin: 0px= 4px;
padding: 4px 6px; border-radius: 2px; } pre { padding: 8px 15px;
background: rgb(25, 25, 25); border-radius: 2px; b= order: 1px solid
rgb(18, 18, 18); box-shadow: rgba(0, 0, 0, 0.3) 0px 1px 3p= x inset;
overflow: auto hidden; } pre code { color: rgb(239, 239, 239);
text-shadow: rgb(0, 0, 0) 0px 1px 0px= ; margin: 0px; padding: 0px; }
table { width: 100%; border-collapse: collapse; } th { text-align: left;
padding: 5px 10px; border-bottom: 1px solid rgb(67, = 67, 67); color:
rgb(182, 182, 182); font-weight: normal; font-family: OpenS=
ansSemibold, \"Helvetica Neue\", Helvetica, Arial, sans-serif
!important; } td { text-align: left; padding: 5px 10px; border-bottom:
1px solid rgb(67, = 67, 67); } hr { border: 0px; outline: none; height:
3px; background: url(\"../images/hr= .gif\") center center repeat-x
transparent; margin: 0px 0px 20px; } dt { color: rgb(240, 231, 213);
font-weight: normal; font-family: OpenSansS= emibold, \"Helvetica
Neue\", Helvetica, Arial, sans-serif !important; } #header { z-index:
100; left: 0px; top: 0px; height: 60px; width: 100%; pos= ition: fixed;
background: url(\"../images/nav-bg.gif\") rgb(53, 53, 53); bord=
er-bottom: 4px solid rgb(67, 67, 67); box-shadow: rgba(0, 0, 0, 0.25)
0px 1= px 3px; } #header nav { max-width: 650px; padding: 0px 10px;
background: blue; margin= : 6px auto; } #header nav ul {
list-style-type: none; margin: 0px; padding: 0px; } #header nav ul li {
font-family: OpenSansLight, \"Helvetica Neue\", Helvetica= , Arial,
sans-serif; font-weight: normal; list-style: none; display: inline= ;
color: white; line-height: 50px; text-shadow: rgba(0, 0, 0, 0.2) 0px 1px
= 0px; font-size: 14px; } #header nav ul li a { color: white; border:
1px solid rgb(93, 145, 11); bac= kground: linear-gradient(rgb(147, 189,
32), rgb(101, 158, 16)) rgb(147, 189= , 32); border-radius: 2px;
box-shadow: rgba(255, 255, 255, 0.3) 0px 1px 0px= inset, rgba(0, 0, 0,
0.7) 0px 3px 7px; padding: 10px 12px; margin-top: 6px= ; line-height:
14px; font-size: 14px; display: inline-block; text-align: ce= nter; }
#header nav ul li a:hover { background: linear-gradient(rgb(116, 150,
25), = rgb(82, 127, 14)) rgb(101, 158, 16); border: 1px solid rgb(82,
127, 14); bo= x-shadow: rgba(0, 0, 0, 0.2) 0px 1px 1px inset,
transparent 0px 1px 0px; } #header nav ul li.fork { float: left;
margin-left: 0px; } #header nav ul li.downloads { float: right;
margin-left: 6px; } #header nav ul li.title { float: right;
margin-right: 10px; font-size: 11px= ; } section { max-width: 650px;
padding: 30px 0px 50px; margin: 70px 0px 20px; = } section #title {
border: 0px; outline: none; margin: 0px 0px 50px; padding:= 0px 0px 5px;
} section #title h1 { font-family: OpenSansLight, \"Helvetica Neue\",
Helvetica= , Arial, sans-serif; font-weight: normal; font-size: 40px;
text-align: cent= er; line-height: 36px; } section #title p { color:
rgb(215, 207, 190); font-family: OpenSansLight, \"= Helvetica Neue\",
Helvetica, Arial, sans-serif; font-weight: normal; font-si= ze: 18px;
text-align: center; } section #title .credits { font-size: 11px;
font-family: OpenSansRegular, \"H= elvetica Neue\", Helvetica, Arial,
sans-serif; font-weight: normal; color: r= gb(105, 105, 105);
margin-top: -10px; } section #title .credits.left { float: left; }
section #title .credits.right { float: right; } \@media print, screen
and (max-width: 720px) { #title .credits { display: block; width: 100%;
line-height: 30px; text-al= ign: center; } #title .credits .left {
float: none; display: block; } #title .credits .right { float: none;
display: block; } } \@media print, screen and (max-width: 480px) {
#header { margin-top: -20px; } section { margin-top: 40px; } nav {
display: none; } } #title { display: block; } time { display: block;
text-align: right; font-style: italic; border-top: 1= px dashed gray; }
div.wrapper { padding: 0px 10px; } audio { width: 100%; margin: 20px
0px; } #header nav { background: none !important; } li \> ul { margin:
0px; } #title + ul { padding-left: 20px; }
\-\-\-\-\--MultipartBoundary\--cq6K1Gf6bh4FhITIQvwBOV2ZLTktzovsSTb1dYK5Lk\-\-\-\-\--
