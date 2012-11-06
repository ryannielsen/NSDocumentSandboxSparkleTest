Quick 'n' dirty project that tests how well a vanilla sandboxed
NSDocument-based app plays with Sparkle, created to track down a
bug we were seeing with Sparkle and the OS X sandbox.

With sandboxed apps, Sparkle needs an XPC service that can be
invoked outside of the app's sandbox, to handle replacing the old
app with the new app. The bug we hit lies in how Sparkle's XPC
service handles its security sessions. By default, all XPC
services create their own independent security session. However,
with NSDocument based applications using Sparkle, this triggers
what appears to be a bug in 10.8.

The newly updated app is launched and inherits the XPC service's
security session. On launch, com.apple.security.pboxd is invoked
(I presume to restore state if the app has documents saved
outside of its sandbox that aren't tracked by security-scoped
bookmarks?). Normally, if the app is in the user's security
session, this is an invisible, unremarkable event. However, in
the special case of the app being launched by Sparkle's XPC
service living in its own security session, this pboxd invocation
actually appears as a running process in the Dock that looks
identical to the newly launched app.

After following a ton of dead ends, it turns out fixing this
issue is quite simple: Sparkle's XPC service needs to declare it
should exist in the invoking app's security session. This is done
quite simply, by setting `JoinExistingSession` to `YES` in the
`XPCService` dictionary of the service's plist. 

Anyway, I figure this project serves as a simple test case for
updating a sandboxed NSDocument-based app using Sparkle.

The Xcode project has two user-defined build settings:
__SPARKLE_BASE_URL__ and __SPARKLE_UPDATE_PATH__. You will likely
need to change __SPARKLE_BASE_URL__; it’s `http://localhost:3000`
because I often test with `adsf` as a local server, and it fires
up on port 3000 by default.

When the Current Project Version is greater than 1, this project
is configured to create a folder named _Appcast_ on your desktop
which hold all of the bits necessary to do a Sparkle update.
Place the contents of this directory in the base url path you
defined above, or kick off a local server like `adsf` in that
newly created directory.


1. Double check that the app target's Current Project Version is
any value greater than 1, and then build
2. Invoke a web server in the newly created ~/Desktop/Appcast
   directory, or upload the contents of that directory to the
   webserver at the path you defined in the
   __SPARKLE_UPDATE_PATH__ user-defined build setting.
3. Clean the project
4. Change the project's Current Project Version build setting to 1
5. Build and run
6. Click the Check For Updates button
