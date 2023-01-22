# snippetbox
Creating a text sharing web application in Go
# Let’s Go! Notes

# Web Application Basics

First you need a handler. Responsible for executing your application logic and for writing HTTP response headers and bodies.

Second, you need a router (or servemux in Go terms). This stores a mapping between the URL patterns for your app and corresponding handlers. Usually you have one servemux for your app containing all your routes.

Third, a web server. Establish a web server and listen for incoming requests as part of the app itself. No 3rd party service like nginx or apache.

Go’s servemux supports two different types of URL patterns: fixed paths and subtree paths.
Fixed paths don’t end with a trailing slash, whereas subtree paths do end with a trailing slash.
Our two new patterns — "/snippet/view" and "/snippet/create" — are both examples of
fixed paths. In Go’s servemux, fixed path patterns like these are only matched (and the
corresponding handler called) when the request URL path exactly matches the fixed path.

So what if you don’t want the "/" pattern to act like a catch-all?
For instance, in the application we’re building we want the home page to be displayed if —
and only if — the request URL path exactly matches "/". Otherwise, we want the user to
receive a 404 page not found response.

It’s not possible to change the behavior of Go’s servemux to do this, but you can include a
simple check in the home hander which ultimately has the same effect

The DefaultServeMux
If you’ve been working with Go for a while you might have come across the http.Handle()
and http.HandleFunc() functions. These allow you to register routes without declaring a
servemux, like this:
func main() {
http.HandleFunc("/", home)
http.HandleFunc("/snippet/view", snippetView)
http.HandleFunc("/snippet/create", snippetCreate)
log.Print("Starting server on :4000")
err := http.ListenAndServe(":4000", nil)
log.Fatal(err)
}
Behind the scenes, these functions register their routes with something called the
DefaultServeMux. There’s nothing special about this — it’s just regular servemux like we’ve
already been using, but which is initialized by default and stored in a net/http global
variable. Here’s the relevant line from the Go source code:
var DefaultServeMux = NewServeMux()
Although this approach can make your code slightly shorter, I don’t recommend it for
production applications.
Because DefaultServeMux is a global variable, any package can access it and register a route
— including any third-party packages that your application imports. If one of those thirdparty packages is compromised, they could use DefaultServeMux to expose a malicious
handler to the web.

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea0abad4-3447-4eba-92e1-e121fe9cce86/Untitled.png)

If you want to send a non-200 status code and a plain-text response body (like we are in the
code above) then it’s a good opportunity to use the http.Error() shortcut. This is a
lightweight helper function which takes a given message and status code, then calls the
w.WriteHeader() and w.Write() methods behind-the-scenes for us.

In terms of functionality this is almost exactly the same. The biggest difference is that we’re
now passing our http.ResponseWriter to another function, which sends a response to the
user for us.
The pattern of passing http.ResponseWriter to other functions is super-common in Go, and
something we’ll do a lot throughout this book. In practice, it’s quite rare to use the w.Write()
and w.WriteHeader() methods directly like we have been doing so far. But I wanted to
introduce them upfront because they underpin the more advanced (and interesting!) ways to
send responses

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/655afa66-90f5-47ec-b4e1-61bc3c3e8e6f/Untitled.png)

# Directory structuring for project

![Untitled](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8ae60dfd-84f7-4f5f-8eb0-2db5104184f6/Untitled.png)

The cmd directory will contain the application-specific code for the executable applications
in the project. For now we’ll have just one executable application — the web application —
which will live under the cmd/web directory.

The internal directory will contain the ancillary non-application-specific code used in the
project. We’ll use it to hold potentially reusable code like validation helpers and the SQL
database models for the project.

The ui directory will contain the user-interface assets used by the web application.
Specifically, the ui/html directory will contain HTML templates, and the ui/static
directory will contain static files (like CSS and images).

It gives a clean separation between Go and non-Go assets. All the Go code we write will
live exclusively under the cmd and internal directories, leaving the project root free to
hold non-Go assets like UI files, makefiles and module definitions (including our go.mod
file). This can make things easier to manage when it comes to building and deploying your
application in the future.
2. It scales really nicely if you want to add another executable application to your project.
For example, you might want to add a CLI (Command Line Interface) to automate some
administrative tasks in the future. With this structure, you could create this CLI application
under cmd/cli and it will be able to import and reuse all the code you’ve written under
the internal directory.

It’s important to point out that the directory name internal carries a special meaning and
behavior in Go: any packages which live under this directory can only be imported by code