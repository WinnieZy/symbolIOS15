# symbolIOS
symbol iOS symbols

# iOS符号化操作步骤

1、iOA.app、iOA.app.dSYM、iOAPacketTunnel.appex.dSYM、.crash文件
2、执行命令，复制出symbolicatecrash可执行文件

```
find /Applications/Xcode.app -name symbolicatecrash -type f
（如/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash）
```

3、uuid确认

```
xcrun dwarfdump --uuid iOA.app/iOA --确认iOA可执行文件uuid
      xcrun dwarfdump --uuid iOA.app.dSYM --确认iOA可执行文件uuid
      xcrun dwarfdump --uuid iOA.app/PlugIns/iOAPacketTunnel.appex/iOAPacketTunnel --确认Packet uuid
      xcrun dwarfdump --uuid iOAPacketTunnel.appex.dSYM --确认Packet dSYM的uuid
      grep --after-context=2 "Binary Images:" *crash --确认crash文件的uuid
```

4、执行环境配置

```
export DEVELOPER_DIR=/Applications/Xcode.app/Contents/Developer
```

5、符号化crash文件

```
./symbolicatecrash xxx.crash xxxl.app.dSYM > out.log
```

## iOS15 symbolicatecrash解析失败

> - To support the new JSON-format crash logs generated in macOS Monterey and iOS 15, Instruments includes a new `CrashSymbolicator.py` script. This Python 3 script replaces the `symbolicatecrash` utility for JSON-format logs and supports inlined frames with its default options. For more information, see: `CrashSymbolicator.py --help`. `CrashSymbolicator.py` is located in the `Contents/SharedFrameworks/CoreSymbolicationDT.framework/Resources/` subdirectory within Xcode 13. (78891800)

iOS 15 之后 Apple 对符号化文件格式进行了 JSON 支持, 所以针对 iOS 15 以上产生的崩溃文件, 写入方式应该是做了调整, 所以在对 iOS 15 以上崩溃文件进行符号化时, 直接使用 CrashSymbolicator.py 来解析, 否则会出现符号化失败, 报错 No crash report version in file 的问题。

```shell
No crash report version in iOA-2022-06-16-173416.ips at ./symbolicatecrash line 1371.
```

**查找CrashSymbolicator.py**

```
find /Applications/Xcode.app -name CrashSymbolicator -type f
```

和使用 symbolicatecrash 方式类似, 先找到其路径, 系统列出不同平台 sh, 切换到最后一个 /Applications/Xcode.app/Contents/SharedFrameworks/CoreSymbolicationDT.framework/Versions/A/Resources

稍微和 symbolicatecrash 不同的是, 其调用方式可以支持参数的方式来排列文件顺序,并且其是用 python 写的脚本, 所以要使用 python3 来进行调用, 否则会报错。

-d '符号表路径' -o '输出符号化路径' -p '苹果给的崩溃日志'

**使用CrashSymbolicator.py**

```
python3 CrashSymbolicator.py -d /dSYMs -o /xxxSymbo.crash -p /xxxCrash.ips
```

