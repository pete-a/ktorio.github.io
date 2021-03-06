---
title: Uploads
caption: Handling HTTP Uploads  
section: Servers
keywords: multipart receiving reading files
permalink: /servers/uploads.html
priority: 1200
---

Ktor supports handling HTTP Uploads. As well as [receiving any other kind of content](/servers/requests.html).

You can check the [Youkube example](/samples/youkube.html) for a full example using.
{: .note.example } 

## Receiving files using multipart

```kotlin
val multipart = call.receiveMultipart()
multipart.forEachPart { part ->
    when (part) {
        is PartData.FormItem -> {
            if (part.name == "title") {
                title = part.value
            }
        }
        is PartData.FileItem -> {
            val ext = File(part.originalFileName).extension
            val file = File(uploadDir, "upload-${System.currentTimeMillis()}-${session.userId.hashCode()}-${title.hashCode()}.$ext")
            part.streamProvider().use { its -> file.outputStream().buffered().use { its.copyTo(it) } }
            videoFile = file
        }
    }

    part.dispose()
}
```