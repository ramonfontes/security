---
tags: security-tutorials
---

# Steganography

**In this short demo you will:** 
- Learn on how to steganography some data.
- Requirements: **steghide**

First of all you need to save the image available at https://zap.aeiou.pt/wp-content/uploads/2017/03/70d00ae2b56f5520f54313168c1636e5.jpg and rename it to `originalimage.jpg`.

Next, create a text file called `message.txt`, add some content and type the following command:

```
$ steghide embed -ef message.txt -cf originalimage.jpg -sf originalimagewithsomedata.jpg
```

Now, open `originalimagewithsomedata.jpg` and take a look at the picture. Where is the content of the `message.txt`file?


Now, let's extract the content of `message.txt` file with the command below.

```
$ steghide extract -sf originalimagewithsomedata.jpg -xf newMessage.txt 
```
