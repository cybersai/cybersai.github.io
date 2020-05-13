---
extends: _layouts.post
section: content
title: Working with XML API
date: 2020-05-13
description: How to send and receive xml data
cover_image: /assets/img/post-cover-image-xml.png
featured: true
categories: [laravel]
---

When you have to integrate a system with an XML API, things can get very messy.
This is because most develops are comfortable with JSON APIs.
Also none of the HTTP clients we love support XML data out of the box.
After working with a few XML APIs for some payment system for clients,
I found some good tips to make working with XML data fun. Let dive in.

> This post assumes you are using Laravel

The first thing we need is an Http Client. You may decide to use curl,
but I prefer not to reinvent the wheels. GuzzleHttp is the best, but it
very cumbersome to use. There is a library called Zttp which is
a wrapper around guzzle for common use cases. For Laravel 7, you don't have to
install anything, other than that you have to install it with the following command.

```bash
composer require kitetail/zttp
```

To use we call the static methods like this:
```php
// Laravel 7+
Http::post('https://api.someapp.io', [
    'name' => 'Jerry'
]);

// Laravel 6 and below
Zttp::post('https://api.someapp.io', [
    'name' => 'Jerry'
]);
```

> Use Zttp or Http were applies to you. From here onwards, we will be using Zttp.
To use this library to send XML data do the following.

```php
Zttp::send('POST', 'https://api.someapp.io', [
    'body' => "<?xml version="1.0"?><note><to>$name</to><from>$user</from><heading>$heading</heading><body>Don't forget me this weekend!</body></note>"
]);
```

When the XML data is much longer, it will not look good You can use heredocs string to make it better.

```php
Zttp::send('POST', 'https://api.someapp.io', [
    'body' => <<<BODY
<?xml version="1.0"?>
<note>
    <to>$name</to>
    <from>$user</from>
    <heading>$heading</heading>
    <body>Don't forget me this weekend!</body>
</note>
    BODY;
]);
```

Wow, this looks much better. Yeah, true, but writing your XML data along
with your code isn't the best. You can separate them to get clean code.
Create a file where you put the XML template inside. The extension of the
file can be .stub

```xml
<!-- xml_data.stub -->
<?xml version="1.0"?>
<note>
    <to>[name]</to>
    <from>[user]</from>
    <heading>[heading]</heading>
    <body>Don't forget me this weekend!</body>
</note>
```

The `[name]`, `[user]`, etc are placeholders to substitute the real data.

In your PHP code, the code becomes as such.

```php
Zttp::send('POST', 'https://api.someapp.io', [
    'body' => preg_replace_array(
        [ '/\[name\]/', '/\[user\]/', '/\[heading\]/', ],
        [ $name, $user, $heading, ],
        file_get_contents(__DIR__.'/xml_data.stub')
    )
]);
```

For this, no matter how bulky the XML data is, you have the privilege of
specifying how it should look and you just swap the placeholders with real
data before sending it.

Hope you found this post helpful, see ya soon.
