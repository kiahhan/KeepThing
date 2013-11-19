Keep Things In Mind 
=========

### How do I run Application(Eclipse or IntelliJ) on Mac OS X with JDK 7?

When running Eclipse Kepler or IDEA 12 on OS X Mavericks with JDK 1.7, I got a message *To open "Eclipse," you need a Java SE 6 runtime. Would you like to install one now?* How to resolve this problem?

This is in part due to Oracle's missing definitions of the JRE7 VM capabilities.
In case you don't want to install JRE6 at all and simply use JRE7 without symlinking it to the JRE6 either you can do the following:
Copy the Info.plist located at the path named below to e.g. ~/Downloads/:

    /Library/Java/JavaVirtualMachines/jdk.1.7.<…>/Contents/

and then replace

```xml  
<key>JVMCapabilities</key>
  <array>
    <string>CommandLine</string>
  </array>
```
    
with the following:

```xml
<key>JVMCapabilities</key>
  <array>
    <string>JNI</string>
    <string>BundledApp</string>
    <string>WebStart</string>
    <string>Applets</string>
    <string>CommandLine</string>
  </array>
```
    
Afterwards copy the file back to its original location (you need administrator rights). For this change to take effect you need to log out of your account (and back in) or restart your computer. The dialog for Java 6 should shouldn't appear anymore and Eclipse should launch just fine using JRE7. The same holds true for any other application that initially asks for Java, e.g. Adobe's Creative Suite.


About [**IDEA IntelliJ 12**][1], there is a little bit complication.
you need to change IDEA IntelliJ JVMVersion to 1.7.x in *Info.plist*, to force running under JDK 1.7.x

> To edit /Applications/<Product>.app/Contents/Info.plist file, change JVMVersion from 1.6.x to 1.7.x :

```xml
<key>JVMVersion</key>
<string>1.7*</string>
```
    
* See this answer for the known problems with JDK 1.7.
* IDEA_JDK environment variable can be used to override the selected JDK, you may need to run the product from the Terminal so that it sees your environment variables (Mac OS limitation): open -a /Applications/<Product>.app/ .
* Application About dialog will show the actual JDK version.

[1]: https://intellij-support.jetbrains.com/entries/23455956-Selecting-the-JDK-version-the-IDE-will-run-under

**Reference link:**

http://stackoverflow.com/questions/19563766/eclipse-kepler-for-os-x-mavericks-request-java-se-6
http://stackoverflow.com/questions/13019199/how-do-i-run-idea-intellij-on-mac-os-x-with-jdk-7

***

