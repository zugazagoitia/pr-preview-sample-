---
layout: tutorial
official: true
title: "Creating a Google Docs clone with WebSockets"
author: <a href="https://www.linkedin.com/in/davidaase" target="_blank">David Åse</a>
date: 2018-04-22
permalink: /tutorials/realtime-collaboration-example
github: https://github.com/javalin/javalin-samples/tree/main/javalin5/javalin-realtime-collaboration-example
summarytitle: WebSockets Google Docs clone
summary: Learn how to create a very basic clone of Google Docs with WebSockets in Javalin
language: ["java", "kotlin"]
---

## What You Will Learn
In this tutorial we will create a very simple realtime collaboration tool (like google docs).\\
We will be using WebSockets for this, as WebSockets provides us with two-way
communication over a one connection, meaning we won't have to
make additional HTTP requests to send and receive messages.
A WebSocket connection stays open, greatly reducing latency (and complexity).

## Dependencies

First we create a Maven project with our dependencies [(→ Tutorial)](/tutorials/maven-setup).\\
We will be using Javalin for our web-server and WebSockets:

```xml
<dependencies>
    <dependency>
        <groupId>io.javalin</groupId>
        <artifactId>javalin-bundle</artifactId>
        <version>{{site.javalinversion}}</version>
    </dependency>
</dependencies>
```

## The Javalin application
The Javalin application is pretty straightforward.
We need:
* a data class (`Collab`) containing the document and the collaborators
* a map to keep track of document-ids and `Collab`s
 * websocket handlers for connect/message/close

We can get the entire server done in about 30-40 lines:

{% capture java %}
import io.javalin.Javalin;
import io.javalin.http.staticfiles.Location;
import io.javalin.websocket.WsContext;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class JavalinRealtimeCollaborationExampleApp {

    private static final Map<String, Collab> collabs = new ConcurrentHashMap<>();

    public static void main(String[] args) {

        Javalin.create(config -> {
            config.staticFiles.add("/public", Location.CLASSPATH);
        }).ws("/docs/{doc-id}", ws -> {
            ws.onConnect(ctx -> {
                if (getCollab(ctx) == null) {
                    createCollab(ctx);
                }
                getCollab(ctx).clients.add(ctx);
                ctx.send(getCollab(ctx).doc);
            });
            ws.onMessage(ctx -> {
                getCollab(ctx).doc = ctx.message();
                getCollab(ctx).clients.stream().filter(c -> c.session.isOpen()).forEach(s -> {
                    s.send(getCollab(ctx).doc);
                });
            });
            ws.onClose(ctx -> {
                getCollab(ctx).clients.remove(ctx);
            });
        }).start(7070);

    }

    private static Collab getCollab(WsContext ctx) {
        return collabs.get(ctx.pathParam("doc-id"));
    }

    private static void createCollab(WsContext ctx) {
        collabs.put(ctx.pathParam("doc-id"), new Collab());
    }

}
{% endcapture %}
{% capture kotlin %}
import io.javalin.Javalin
import io.javalin.http.staticfiles.Location
import io.javalin.websocket.WsContext
import java.util.concurrent.ConcurrentHashMap

fun main() {

    val collaborations = ConcurrentHashMap<String, Collaboration>()

    Javalin.create {
        it.staticFiles.add("/public", Location.CLASSPATH)
    }.apply {
        ws("/docs/{doc-id}") { ws ->
            ws.onConnect { ctx ->
                if (collaborations[ctx.docId] == null) {
                    collaborations[ctx.docId] = Collaboration()
                }
                collaborations[ctx.docId]!!.clients.add(ctx)
                ctx.send(collaborations[ctx.docId]!!.doc)
            }
            ws.onMessage { ctx ->
                collaborations[ctx.docId]!!.doc = ctx.message()
                collaborations[ctx.docId]!!.clients.filter { it.session.isOpen }.forEach {
                    it.send(collaborations[ctx.docId]!!.doc)
                }
            }
            ws.onClose { ctx ->
                collaborations[ctx.docId]!!.clients.remove(ctx)
            }
        }
    }.start(7070)

}

val WsContext.docId: String get() = this.pathParam("doc-id")
{% endcapture %}
{% include macros/docsSnippet.html java=java kotlin=kotlin %}

We also need to create a data object for holding our document and the people working on it:

{% capture java %}
import io.javalin.websocket.WsContext;
import java.util.Set;
import java.util.concurrent.ConcurrentHashMap;

public class Collab {
    public String doc;
    public Set<WsContext> clients;

    public Collab() {
        this.doc = "";
        this.clients = ConcurrentHashMap.newKeySet();
    }
}
{% endcapture %}
{% capture kotlin %}
data class Collaboration(var doc: String = "", val clients: MutableSet<WsContext> = ConcurrentHashMap.newKeySet())
{% endcapture %}
{% include macros/docsSnippet.html java=java kotlin=kotlin %}

## Building a JavaScript Client
In order to demonstrate that our application works, we can build a JavaScript client.
We'll keep the HTML very simple, we just need a heading and a text area:

```markup
<body>
    <h1>Open the URL in another tab to start collaborating</h1>
    <textarea placeholder="Type something ..."></textarea>
</body>
```

The JavaScript part could also be very simple, but we want some slightly advanced features:

* When you open the page, the app should either connect to an existing document or generate a new document with a random id
* When a WebSocket connection is closed, it should immediately be reestablished
* When new text is received, the user caret ("text-cursor") should remain in the same location (easily the most complicated part of the tutorial).

```javascript
window.onload = setupWebSocket;
window.onhashchange = setupWebSocket;

if (!window.location.hash) { // document-id not present in url
    const newDocumentId = Date.now().toString(36); // this should be more random
    window.history.pushState(null, null, "#" + newDocumentId);
}

function setupWebSocket() {
    const textArea = document.querySelector("textarea");
    const ws = new WebSocket(`ws://localhost:7070/docs/${window.location.hash.substr(1)}`);
    textArea.onkeyup = () => ws.send(textArea.value);
    ws.onmessage = msg => { // place the caret in the correct position
        const offset = msg.data.length - textArea.value.length;
        const selection = {start: textArea.selectionStart, end: textArea.selectionEnd};
        const startsSame = msg.data.startsWith(textArea.value.substring(0, selection.end));
        const endsSame = msg.data.endsWith(textArea.value.substring(selection.start));
        textArea.value = msg.data;
        if (startsSame && !endsSame) {
            textArea.setSelectionRange(selection.start, selection.end);
        } else if (!startsSame && endsSame) {
            textArea.setSelectionRange(selection.start + offset, selection.end + offset);
        } else { // this is what google docs does...
            textArea.setSelectionRange(selection.start, selection.end + offset);
        }
    };
    ws.onclose = setupWebSocket; // should reconnect if connection is closed
}
```

And that's it! Now try opening `localhost:7070` in a couple of different
browser windows (that you can see simultaneously) and collaborate with yourself.

## Conclusion
We have a working realtime collaboration app written in less than 100 lines of Java and JavaScript.
It's very basic though, some things to add could include:

* Show who is currently editing the document
* Persist the data in a database at periodic intervals
* Replace the textarea with a rich text editor, such as [quill](https://quilljs.com)
* Replace the textarea with a code editor such as [ace](https://ace.c9.io/) for collaborative programming
* Improving the collaborative aspects with [operational transformation](https://en.wikipedia.org/wiki/Operational_transformation)

The use cases are not limited to text and documents though, you should use WebSockets
for any project which requires a lot of interactions with low latency. Have fun!
