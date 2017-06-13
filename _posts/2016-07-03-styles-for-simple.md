---
layout: post
title:  Fast Styles
category: tools 
description: fast styles easy to search
---

Simple is a beautiful but functional jekyll theme. The font-type setting looks really good when writers use CJK mixed with English.

[link](http://github.com/wild-flame/jekyll-simple)
```
[link](http://github.com/wild-flame/jekyll-simple)
```

**strong**
```
**strong**
```

*italic*
```
*italic*
```

~~deletion~~
```
~~deletion~~
```

<ins>insertion</ins>
```
<ins>insertion</ins>
```


### Headers:

# Header 1
```
# Header 1
```

## Header 2
```
## Header 2
```

### Header 3
```
### Header 3
```

#### Header 4
```
#### Header 4
```

##### Header 5
```
##### Header 5
```

###### Header 6
```
###### Header 6
```

### Lists:

- list item 1
- list item 2
- list item 3
```
-list item 1
-list item 2
-list item 3
```

1. list item 1
2. list item 2
3. list item 3
```
1.list item 1
2.list item 2
3.list item 3
```

### Blockquote:

> Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.

```
> Lorem ipsum dolor sit amet...
```
### [BASSCSS](http://www.basscss.com/) colors:

- <span class="black">black</span>
- <span class="gray">gray</span>
- <span class="silver">silver</span>
- <span class="white">white</span>
- <span class="aqua">aqua</span>
- <span class="blue">blue</span>
- <span class="navy">navy</span>
- <span class="teal">teal</span>
- <span class="green">green</span>
- <span class="olive">olive</span>
- <span class="lime">lime</span>
- <span class="yellow">yellow</span>
- <span class="orange">orange</span>
- <span class="red">red</span>
- <span class="fuchsia">fuchsia</span>
- <span class="purple">purple</span>
- <span class="maroon">maroon</span>
```
<span class="black">black</span>
<span class="gray">gray</span>
<span class="silver">silver</span>
<span class="white">white</span>
<span class="aqua">aqua</span>
<span class="blue">blue</span>
<span class="navy">navy</span>
<span class="teal">teal</span>
<span class="green">green</span>
<span class="olive">olive</span>
<span class="lime">lime</span>
<span class="yellow">yellow</span>
<span class="orange">orange</span>
<span class="red">red</span>
<span class="fuchsia">fuchsia</span>
<span class="purple">purple</span>
<span class="maroon">maroon</span>
```

### Horizontal rule:

-----------------------
```
-----------------------
```

### Image:

![]({{site.baseurl}}/assets/img/image.jpg)
```
![]({{site.baseurl}}/assets/img/image.jpg)
```

### Table:


| Default aligned |Left aligned| Center aligned  | Right aligned  |
|-----------------|:-----------|:---------------:|---------------:|
| First body part |Second cell | Third cell      | fourth cell    |
| Second line     |foo         | **strong**      | baz            |
| Third line      |quux        | baz             | bar            |
| Second body     |            |                 |                |
| 2 line          |            |                 |                |
| Footer row      |            |                 |                |

| left | center | right |
| :--- | :----: | ----: |
| aaaa | bbbbbb | ccccc |
| a    | b      | c     |
```
| left | center | right |
| :--- | :----: | ----: |
| aaaa | bbbbbb | ccccc |
| a    | b      | c     |
```

### Code snippet

```javascript
// index.js
var arr = [1, 2, 3, 4, 5];
var b = arr.map(x => x * x);
console.log(b);

function foo(){
	console.log('foo');
}

```
