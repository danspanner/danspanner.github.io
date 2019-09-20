---
layout: post
title: Stuff to do with a Raspberry Pi
---

I went down a bit of a rabbit hole to see if I could reverse out the photo from AD.

https://docs.microsoft.com/en-us/windows/win32/adschema/a-photo

Turns out, AD can store a user photo as a byte stream. In AD, if you view the attribute, you will see the raw hex of the photo. We can tell it's a JPEG from the FF D8

https://github.com/corkami/pics/blob/master/binary/JPG.png

So how do we get it out? With our good friend system.io.file!

First, we'll store the properties for our user in a variable.

`$photo = get-aduser -Identity t.smith -Properties jpegPhoto`

Then

`[system.io.file]::writeallbytes('C:\tmp\output.jpg', $photo.jpegPhoto.value)`

