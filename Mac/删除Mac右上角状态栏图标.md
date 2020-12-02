有的时候我们删除某个应用程序后，发现在右上角的状态栏还有图标显示，而且每次开机启动，右上角的图标也都在很是可恶。

每次开机启动，右上角状态栏的图标也跟着启动，此类型的启动项都由launchd来负责启动，launchd是Mac OS下用于初始化系统环境的关键进程，它是内核装载成功之后在OS环境下启动的第一个进程。采用这种启动方式的配置也很简单，只需要一个plist文件，该plist文件存在的目录有：

```
~/Library/LaunchAgents

/Library/LaunchAgents

/System/Library/LaunchAgents
```

我们只需分别前往这三个目录，找到对应的plist文件，例如：
**com.malwarebytes.mbam.frontend.agent.plist**
打开这个文件，查看到的内容：

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
	<key>Label</key>
	<string>com.malwarebytes.mbam.frontend.agent</string>
	<key>Program</key>
	<string>/Library/Application Support/Malwarebytes/MBAM/Engine.bundle/Contents/PlugIns/FrontendAgent.app/Contents/MacOS/FrontendAgent</string>
	<key>LimitLoadToSessionType</key>
	<array>
		<string>Aqua</string>
	</array>
	<key>KeepAlive</key>
	<true/>
	<key>RunAtLoad</key>
	<true/>
	<key>EnableTransactions</key>
	<false/>
	<key>ExitTimeOut</key>
	<integer>3</integer>
</dict>
</plist>
```

可以找到这个图标的程序所在目录，前往这个目录，删除文件夹；同时删除这个plist文件，再重新启动Mac。

这样，这个状态栏图标就消失了。
