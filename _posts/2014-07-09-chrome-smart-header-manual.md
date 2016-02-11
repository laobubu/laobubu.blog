---
layout: post
title:  "Smart Header Manual"
date:   2014-7-9 17:19:00
categories: ["Manual", "Doc"]
permalink: /archives/chrome-smart-header-manual/
excerpt_separator: <!--more-->
---

![Smart Header](http://i671.photobucket.com/albums/vv73/laobubu/pics%20for%20blog/banner/banner1.jpg)

This is the official manual of [Smart Header](https://chrome.google.com/webstore/detail/smart-header/ncgnmldbedmbadafajhjeahmafdmggbp), providing some essential guides. If you have any question, please <a href="#comment-form">leave a comment</a> and I will answer via email.

<!--more-->

## Basic Usage

### Change a header's value

1. Click the Smart Header Icon on your Chrome tool-bar.
2. Choose the value you want. You can see these options: 
    * **Automatic** : the value will follow your valid AutoRules ,or not change.
    * **Browser's default value** :  nothing will be changed.
    * **Remove** : this header will NOT be sent.
    * **Blank** : this header will be sent with no content.
    * **![Pen button](https://cdn4.iconfinder.com/data/icons/miu/22/editor_pencil_pen_edit_write-16.png)** : input the value manually.
    * *other preset values*

### Add a header

If the headers you need aren't there, just add them.

1. Open the configuration center.
2. Click ![(+) Add button](https://cdn4.iconfinder.com/data/icons/miu/22/circle_add_plus-32.png) on the left.
3. Input the header's name like `X-Forwarded-For`.

### Edit the preset values

1. Open the configuration center.
2. Choose one header on your left.
3. *(optional) Switch to **Presets & Other** tab.*
4. Just edit the values. The format is like
```
Name1=Value1
Name2=Value2
```

### Remove a header from the list

Wont change some headers? Just remove them.

1. Open the configuration center.
2. Choose one of the headers you want to remove.
3. *(optional) Switch to **Presets & Other** tab.*
4. Click **[Delete this header]** .

## Advanced Configuring 

### Automatic Rule (a.k.a AutoRule)

As you can see when you choose a header, there is a tab named **Automatic Rules**. This feature makes Smart Header much more smarter than other extensions.

A header can own many autorules. Before a HTTP request is sent, Smart Header will check the autorules one by one. When an autorule is valid (means that ALL of its conditions are TRUE), this valid autorule will change the header's value and its following autorule will not work.

An autorule can own many conditions. When ALL of its conditions are TRUE, this autorule can work. 

A (basic) condition is consist of ...

* **Source** : the value to be tested.
 * **URL** : the requesting URL.
 * **Referer** : the referer. If it's null, this autorule will not work
 * **Method** : `GET`, `POST`, `PUT` etc.
 * **Req Type** : the type of this request. This can be `main_frame`, `sub_frame`, `stylesheet`, `script`, `image`, `object`, `xmlhttprequest`, or `other`.
* **Method** : the method to test the source value.
* **Argument** : the argument of the method.

Miracle comes from automation!

### Utils

#### RegExp Tester

This is designed for users who want to add an autorule based on regExp. You can test whether your regExp works.

### Master Mode

This is hidden in the **Options** page. You can export & import headers and AutoRules with this. There are 3 buttons under the input area.

 * **Load** : read the headers and autorules from current page.
 * **Write** : replace the header list and autorules with your input.
 * **Mix** : compare current data with your input and append the new things that we found in your input.

### Magic Variables

You can put magic variables like `{rand:0,10}` into your new value and Smart Header will replace it with something like `7`.

#### General Magic Variables

Name          | Description     |  Example Value 
:-------      | :-------------- |:------------------------------
rand:`A`,`B`  | A random integer number from [A, B]  | `{rand:0,10}` = 6 
date:`DateFormat` | A date string. | `{date:yyyy-M-dd}` = 2014-7-09 
url        | URL | `http://laobubu.net/page?test`
scheme        | Request Scheme   |`http`
host        | The host of your URL | `laobubu.net`
path        | The URI without query string | `/page`
query        | The query string | `?test`

##### DateFormat

Sign       | Description                 | Sign      | Description               
:----------|:-----------------------     |:----------|:-----------------------
yy / yyyy  | Year                        | M / MM    | Month (1-12)
d / dd     | Day (1-31)                  | h / hh    | Hours
m / mm     | Minutes                     | s / ss    | Seconds
S          | Milliseconds

<a name="armv"></a>
#### AutoRule Magic Variables

If one autorule contains RegExp and works, it will generate magic variables `{result:#.#}`.

The first number sign(#) is the index of the condition (0,1,2...). When your cursor hovers over one condition, the index will show up.

The second is the index of the capturing group (0,1,2...). Notice that the whole regular expression is one and the first capturing group.

After replacing the signs, your variable reference should be like `{result:1.0}`.

You can test with *Utils - RegExp Tester*.

## FAQ

### My change doesn't work.

* FORCE Refresh the page by hitting Ctrl+F5.
* Clean cache and refresh.
* Clean cookies and refresh.
* Use lower-case name (`X-Forwarded-For` -> `x-forwarded-for` , for example).
* Check if your request header/value is correct.
* Check if the website supports the change.
* For the sake of safety, Chrome limited some headers (e.g. `x-forwarded-for`). There is no way to bypass yet.
* If it still doesn't work, tell me the detail.

### Can I import or export my header list?

Yes. Read the part about Master Mode .

### What's the range that my changes will affect.

Every request in Chrome. I don't know whether there are people want to limit this, therefore I didn't code.

### My question not found.

Leave a comment and I will answer. Remember to leave your real e-mail address and a notification be sent when I answer. Your e-mail address is safe here.