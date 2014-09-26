For the last two months, I worked on making a Mac client that is similar to the dropbox or box client. The idea was to eventually have a windows client too but since the it works in the same way across OS, I decided to have common python code for all the functionality with OS specific python code for the UI. I decided to use PyObjC for writing the Mac specific UI code. 

PyObjC is a bidirectional bridge between the Python and Objective-C programming languages, allowing programmers to use and extend existing Objective-C libraries, such as Apple's Cocoa framework, using Python. While it has all the advantages that python has (rapid prototyping, High-level native data types, ducktyping etc), it also has a few drawbacks. The main drawback is the fact that the documentation is absolutely horrendous.

Let me quickly show you a simple example :

Code to use native mac notifications:

``` 

class Notification(NSObject):

    def notify(self, title, subtitle, text, url):
        notification = NSUserNotification.alloc().init()
        notification.setTitle_(str(title))
        notification.setSubtitle_(str(subtitle))
        notification.setInformativeText_(str(text))
        notification.setSoundName_("NSUserNotificationDefaultSoundName")
        notification.setUserInfo_({"action": "open_url", "value": url})
        NSUserNotificationCenter.defaultUserNotificationCenter().setDelegate_(self)
        NSUserNotificationCenter.defaultUserNotificationCenter().scheduleNotification_(notification)

    def userNotificationCenter_didActivateNotification_(self, center, notification):
        #callback
        userInfo = notification.userInfo()
        print 'came here'
        print userInfo
```

The above code uses the python equvivalent of the iOS [NSNotification](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSNotification_Class/Reference/Reference.html "NSNotification"). Notice that all the : in objective c are replace with _ in pyobjc. This is one very frustrating difference between the two.

Now that we have the Notification class let us use it.

```
class MyApp(NSApplication):

    def finishLaunching(self):
        # Make statusbar item
        statusbar = NSStatusBar.systemStatusBar()
        self.statusitem = statusbar.statusItemWithLength_(NSVariableStatusItemLength)
        self.icon = NSImage.alloc().initByReferencingFile_('icon.png')
        self.icon.setScalesWhenResized_(True)
        self.icon.setSize_((20, 20))
        self.statusitem.setImage_(self.icon)

        # make the menu
        self.menubarMenu = NSMenu.alloc().init()

        self.menuItem = NSMenuItem.alloc().initWithTitle_action_keyEquivalent_('Test Notification', 'clicked:', '')
        self.menubarMenu.addItem_(self.menuItem)

        self.quit = NSMenuItem.alloc().initWithTitle_action_keyEquivalent_('Quit', 'terminate:', '')
        self.menubarMenu.addItem_(self.quit)

        # add menu to statusitem
        self.statusitem.setMenu_(self.menubarMenu)
        self.statusitem.setToolTip_('My App')
        self._notify_obj = Notification.alloc().init()

    def clicked_(self, notification):
        self._notify_obj.notify('Title', 'subtitle', 'description text', 'http://google.com')

```

In the above code, I have created a simple app which will have 2 elements in the menu bar. The first of those items will display a notification which I showed in the previous snippet. The second option is to quit the app.

The app is run using the follwing code:

```
if __name__ == "__main__":
    app = MyApp.sharedApplication()
    AppHelper.runEventLoop()


```

And thats it! We just run this file like any other python file. We will also need an icon.png which will be used as the app icon. The whole project can be found at [Python UI](https://github.com/rvpradeep/Python-UI "Python UI").

The example shown above is a very trivial one. Most applications will need a more elaborate UI. In these cases it might be necessary to use the interface builder in Xcode and then link it to pyobjc variables using appropriate decorators. An example for this can be found at [Simple Login page](https://github.com/rvpradeep/PyObjC-LoginPage "Simple login page"). In this method, we will need to use py2app to make an executable. 




