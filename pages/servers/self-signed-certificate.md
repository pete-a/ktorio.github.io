---
title: Self-Signed Certificate
caption: Using a Self-Signed Certificate  
section: Servers
keywords: self-signed certificate https
permalink: /servers/self-signed-certificate.html
---

Ktor allows you to create and use self-signed certificates for serving HTTPS or HTTP/2 requests.

Right now it is not possible to create self-signed certificates using Java9. Please use Java8 in the meantime.
{: .note}

**Table of contents:**

* TOC
{:toc}

To create a self-signed certificate using Ktor, you have to call the `generateCertificate` function.

```
io.ktor.util.generateCertificate(File("mycert.jks"))
```

Since Ktor requires the certificate when it starts, you have to create the certificate before starting the server.

## Create the certificate using gradle

One possible option is to execute a main class generating the certificate before actually running the server:

### `CertificateGenerator.kt`

You can declare a class with a main method that just generates the certificate when it doesn't exists:

```kotlin
package io.ktor.samples.http2

import io.ktor.application.*
import io.ktor.features.*
import io.ktor.http.*
import io.ktor.response.*
import io.ktor.routing.*
import io.ktor.util.*
import java.io.*

object CertificateGenerator {
    @JvmStatic
    fun main(args: Array<String>) {
        val jksFile = File("build/temporary.jks").apply {
            parentFile.mkdirs()
        }

        if (!jksFile.exists()) {
            generateCertificate(jksFile) // Generates the certificate
        }
    }
}

```

### `build.gradle`

In your `build.gradle` file you can make the `run` task to depend on a `generateJks` task that executes the main
class generating the certificate. For example:

```groovy
task generateJks(type: JavaExec, dependsOn: 'classes') {
    classpath = sourceSets.main.runtimeClasspath
    main = 'io.ktor.samples.http2.CertificateGenerator'
}

getTasksByName("run", false).first().dependsOn('generateJks')
```

## The HOCON `application.conf` configuration file

When creating your HOCON configuration file, you have to add the `ktor.deployment.sslPort`, and the `ktor.security.ssl`
properties to define the ssl port and the keyStore:

`resources/application.conf`:
```groovy
ktor {
    deployment {
        port = 8080
        sslPort = 8443
        watch = [ http2 ]
    }

    application {
        modules = [ io.ktor.samples.http2.Http2ApplicationKt.main ]
    }

    security {
        ssl {
            keyStore = build/temporary.jks
            keyAlias = mykey
            keyStorePassword = changeit
            privateKeyPassword = changeit
        }
    }
}
```

## Ktor normal module

After that you can just write a normal plain Ktor module: 

```kotlin
package io.ktor.samples.http2

import io.ktor.application.*
import io.ktor.features.*
import io.ktor.http.*
import io.ktor.response.*
import io.ktor.routing.*
import io.ktor.util.*
import java.io.*

fun Application.main() {
    install(DefaultHeaders)
    install(CallLogging)
    install(Routing) {
        get("/") {
            call.push("/style.css")

            call.respondText("""
                <!DOCTYPE html>
                <html>
                    <head>
                        <link rel="stylesheet" type="text/css" href="/style.css">
                    </head>
                    <body>
                        <h1>Hello, World!</h1>
                    </body>
                </html>
            """.trimIndent(), contentType = ContentType.Text.Html)
        }

        get("/style.css") {
            call.respondText("""
                h1 { color: olive }
            """, contentType = ContentType.Text.CSS)
        }
    }
}
```
{: .compact }

## Accessing to your server

Then you can point to <https://127.0.0.1:8443/> to access to your server.
Since this is a self-signed certificate, your browser will probably warn you about an invalid certificate, so
you will have to disable that warning.

## Full example

Ktor has a full example using self-signed certificates here:

<https://github.com/ktorio/ktor/tree/08b173e02fe9a9dbee39f48e7162e6ea7a1f8b16/ktor-samples/ktor-samples-ssl-http2>