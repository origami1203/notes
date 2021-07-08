```properties
gradle wrapper
|____gradle
| |____wrapper
| | |____gradle-wrapper.jar  //gradle-wrapper.jar是Gradle Wrapper的主体功能包（wrapper 的代码所在）
| | |____gradle-wrapper.properties  // 配置文件,下载gradle的地址及版本，下载后zip的存储位置，解压后存储位置等，使用gradlew命令的使用，会根据这个文件来使用对应的gradle进行构建
|____gradlew  //Linux 下可执行脚本
|____gradlew.bat  //Windows 下可执行脚本
```

当从版本库下载代码之后，如果你本机安装过gradle，当然直接直接编译运行既可。但是对没有安装gradle的用户，可以执行项目根目录下的`gradlew.bat`脚本（Linux是执行`gradlew`命令），将会在`gradle-wrapper.properties`中的`~/.gradle/wrapper/dists`目录中首次下载并安装gradle并可以编译代码，一个指令可以下载并安装gradle来构建项目。