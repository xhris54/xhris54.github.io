---
title: Site Isolation in Modern Browsers
---



<div align="center">
  <img src="https://i.pinimg.com/originals/f9/57/6f/f9576fca9fc8ef79976a1d6327bbe9ae.gif" alt="Pixel Night Street" width="110%" height="100%" style="border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.3);" />
</div>
 

 
--- 

Hey! I'm Krishna, a curious mind in the world of cybersecurity . In this blog, we will explore the real security impact of Site Isolation in modern browsers, and see exactly how it works under the hood.

Site Isolation is a security architecture feature in Chromium-based browsers like Chrome, Edge, and Brave. Its main purpose is to make sure that different websites run in completely separate renderer processes, even if you open them in the same browser tab or window.

A renderer process is the isolated environment that handles everything inside a web page — it parses the HTML, runs JavaScript, applies CSS, and paints what you see on the screen. By isolating sites into different processes, Site Isolation makes it much harder for one site to interfere with or steal data from another, even if there’s a bug like Spectre or a memory leak.

In simple terms, it’s like putting every site in its own locked room, so they can’t peek at each other’s secrets.

##### Useful Pages
- `chrome://process-internals/` → Shows how frames and processes are allocated.
- `chrome://flags` → Lets you modify browser features — here, we will disable Site Isolation to see the difference.

First, visit `chrome://process-internals/` to see how Chrome splits processes by site and observe the Renderer Process count which is '2' .
<br>

<div align="center">
  <img src="/assets/1.png" alt="Pixel Night Street" width="110%" height="100%" style="border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.3);" />
</div>
 


For this practical, we will use:

1. `127.0.0.1:8000` → a local HTML page.
2. `https://api.defhawk.com/community/category-featured` → remote JSON API.    

**Example local page:**  
`127.0.0.1:8000/test.html`

```
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Iframe Test - Defhawk API</title>
</head>
<body>
  <h1>Iframe Test: Defhawk API JSON</h1>

  <iframe id="apiFrame" src="https://api.defhawk.com/community/category-featured" width="100%" height="500"></iframe>

  <script>
    setTimeout(() => {
      const iframe = document.getElementById('apiFrame');
      try {
        const data = iframe.contentWindow.document.body.innerText;
        console.log('Trying to read iframe content:', data);
      } catch (e) {
        console.error('Blocked by SOP:', e);
      }
    }, 2000);
  </script>
</body>
</html>
```

Now lets load `127.0.0.1:8000/test.html` and check `chrome://process-internals/`.


<div align="center">
  <img src="/assets/2.png" alt="Pixel Night Street" width="110%" height="100%" style="border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.3);" />
</div>
 

<br>


<div align="center">
  <img src="/assets/3.png" alt="Pixel Night Street" width="110%" height="100%" style="border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.3);" />
</div>
 

We can observe:

- Local page → 1 renderer process.
- Cross-origin iframe (api.defhawk.com) → separate renderer process.

Renderer process count goes from `+2` → total `5`.
This confirms that the main page and the cross-origin iframe are isolated.

Now lets disable the site isolation 
Go to `chrome://flags` → search for “Site Isolation” → **disable** it.  
Restart Chrome.

<br>
Now reload the same local page and check `chrome://process-internals/` again.

<div align="center">
  <img src="/assets/4.png" alt="Pixel Night Street" width="110%" height="100%" style="border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.3);" />
</div>
 


This time, you should see:

- The renderer process count only increases by +1 → total 3.
- The main page and the cross-origin iframe now share the same renderer process , means the api.defhawk.com and 127.0.0.1 in same rendrer process 

We can also verify it form chrome://process-internals/#web-contents 
<br>


<div align="center">
  <img src="/assets/5.png" alt="Pixel Night Street" width="110%" height="100%" style="border-radius: 12px; box-shadow: 0 4px 12px rgba(0,0,0,0.3);" />
</div>
 



### Conclusion

With Site Isolation enabled, each site runs in its own renderer process, making it much harder for one site to access or leak data from another site’s memory. This helps protect sensitive information (like session tokens, credentials, or personal data) against modern CPU side-channel attacks, such as Spectre.

When Site Isolation is disabled, unrelated sites can share the same renderer process. This means if a malicious site finds a way to exploit a speculative execution bug, it could potentially read memory belonging to another site loaded in the same process — bypassing traditional Same-Origin Policy protections at the hardware level.

In a real attack, this could allow:

- Leaking cookies or tokens stored in RAM.
- Reading sensitive API responses that were never exposed to the DOM.
- Combining leaked data with XSS or other bugs for more advanced compromise.

SOP and CORS protect logical access, but Site Isolation protects against physical RAM-level leaks when hardware bugs like Spectre exist.

Thanks for visiting.  
Let’s explore the web’s cracks — one request at a time. 

---

*Posted by Krishna | [xhris54.github.io](https://xhris54.github.io)*
