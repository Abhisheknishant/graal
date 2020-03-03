GraalVM supports debugging of guest language applications and provides a
built-in implementation of
the [Chrome DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/).
This allows you to attach compatible debuggers such as
[Chrome Developer Tools](https://developers.google.com/web/tools/chrome-devtools/)
to GraalVM.

To debug guest language applications, pass the `--inspect` option to the
command-line launcher, as in the following
example with a Node.js hello world program:

```
var http = require('http');

var server = http.createServer(function (request, response) {
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.end("Hello World!\n");
});

server.listen(8000);

console.log("Server running at http://localhost:8000/");
```

1. Save this program as `HelloWorld.js` and then run:

```
$ node --inspect --jvm HelloWorld.js
Debugger listening on port 9229.
To start debugging, open the following URL in Chrome:
    chrome-devtools://devtools/bundled/js_app.html?ws=127.0.1.1:9229/76fcb6dd-35267eb09c3
Server running at http://localhost:8000/
```

2. Navigate to `http://localhost:8000/` in your browser to launch the node application.

3. Open the `chrome-devtools:...` link in a separate Chrome browser tab.

4. Navigate to the `HelloWorld.js` file and submit a breakpoint at line 4.

5. Refresh the node.js app and you can see the breakpoint hit.

You can inspect the stack, variables, evaluate variables and selected expressions
in a tooltip, and so on. By hovering a mouse over the `response` variable, for
instance, you can inspect its properties as can be seen in the screenshot below:

![](/docs/img/ChromeInspector.png "Chrome DevTools debugging node in GraalVM")

Consult the
[JavaScript Debugging Reference](https://developers.google.com/web/tools/chrome-devtools/javascript/reference)
for details on Chrome DevTools debugging features.

This debugging process applies to all guest languages that GraalVM supports.
Other languages such as R and Ruby can be debugged as easily as JavaScript,
including stepping through language boundaries during guest language
[interoperability]({{ "/docs/reference-manual/polyglot/" | relative_url }}).

### Inspect Options

### Node Launcher
The node.js implementation that GraalVM provides accepts the same options as
[node.js built on the V8 JavaScript engine](https://nodejs.org/), such as:

```
--inspect[=[host:]<port number>]
```

Enables the inspector agent and listens on port 9229 by default. To listen on a
different port, specify the optional port number.

```
--inspect-brk[=[host:]<port number>]
```

Enables the inspector agent and suspends on the first line of the application
code. Listens on port 9229 by default, to listen on a different port, specify
the optional port number. This applies to the `node` launcher only.

### Other Language Launchers

Other guest language launchers such as `js`, `python`, `Rscript`, `ruby`, `lli` and `polyglot`
accept the `--inspect[=[host:]<port number>]` option, but suspend on the first line of
the application code by default.

```
--inspect.Suspend=(true|false)
```

Disables the initial suspension if you specify `--inspect.Suspend=false`.

### Additional Common Inspect Options
All launchers accept also following additional options:

```
--inspect.Path=<path>
```

Allows to specify a fixed path that generates a predictable connection URL. By
default, the path is randomly generated.

```
--inspect.SourcePath=<source path>
```
This option specifies a list of directories or ZIP/JAR files representing the source path. When the inspected application contains relative references to source files, their content is loaded from locations resolved with respect to this source path. It is useful during LLVM debugging, for instance.
The paths are delimited by `:` on UNIX systems and by `;` on MS Windows.

```
--inspect.Secure=(true|false)
```

When true, use TLS/SSL to secure the debugging protocol. Besides changing the WS
(web socket) protocol to WSS, the HTTP endpoint that serves metadata about the debuggee
is also changed to HTTPS. This is not compatible e.g., with
[chrome://inspect](chrome://inspect) page, which is not able to provide the debuggee
information and launch the debugger then. Launch debugging via the printed WSS URL directly.

Use the standard `javax.net.ssl.*` system options to provide information about
keystore with the TLS/SSL encryption keys, or following options:
* `--inspect.KeyStore` keystore file path,
* `--inspect.KeyStoreType` keystore file type (defaults to JKS),
* `--inspect.KeyStorePassword` keystore password,
* `--inspect.KeyPassword` password for recovering keys, if it's different from the keystore password.

```
--inspect.WaitAttached=(true|false)
```

When true, no guest language source code is executed until the inspector client
is attached. Unlike `--inspect.Suspend=true`, the execution is resumed right
after the client is attached. That assures that no execution is missed by the
inspector client. It is `false` by default.

### Advanced Debug Options
Following options are for language experts and language developers:

```
--inspect.Initialization=(true|false)
```

When true, inspect the language initialization phase. When initial suspension is
active, suspends at the beginning of language initialization and not necessarily
at the beginning of the application code. It's `false` by default.

```
--inspect.Internal=(true|false)
```

When true, internal sources are inspected as well. Internal sources may provide
language implementation details. It's `false` by default.

### Programmatic Launch of Inspector Backend

Embedders can provide the appropriate inspector options to the `Engine/Context`
to launch the inspector backend. The following code snippet provides an example of
a possible launch:

```
String port = "4242";
String path = "session-identifier";
String remoteConnect = "true";
Context context = Context.newBuilder("js")
            .option("inspect", port)
            .option("inspect.Path", path)
            .option("inspect.Remote", remoteConnect)
            .build();
String hostAdress = "localhost";
String url = String.format(
            "chrome-devtools://devtools/bundled/js_app.html?ws=%s:%s/%s",
            hostAdress, port, path);
// Chrome Inspector client can be attached by opening the above url in Chrome
```
