## 构建包含javaFx的jre

1. 下载解压javafx-sdk和javafx-jmods
```shell
# 设置环境变量(当然也可以不设置, 在使用该环境变量的地方用具体路径替代)
set JFX17_PATH="F:\tools\javafx-sdk-17.0.9\lib"
set JMODS17_PATH="F:\tools\javafx-jmods-17.0.9"
```

2. 检查 JavaFX 模块的声明
```shell
java --module-path %JFX17_PATH% --describe-module javafx.base
java --module-path %JFX17_PATH% --describe-module javafx.controls
```

3. 生成包含完整JavaFX的jre
```shell
# 生成包含 java.se 和 JavaFX 全部模块的 JRE
jlink --module-path %JMODS17_PATH% --add-modules java.se,javafx.base,javafx.controls,javafx.fxml,javafx.graphics,javafx.media,javafx.swing,javafx.web --output jre
# 根据依赖传递申明, 可简写为
jlink --module-path %JMODS17_PATH% --add-modules java.se,javafx.web,javafx.swing,javafx.fxml --output jre
```

通过这种方式生成的 JRE, 已经包含了 JavaFX 相关依赖模块, 可以用来直接运行 JavaFX 应用程序, 可以分发给其他用户, 而其他用户不需要再下载 JavaFX SDK / JMods

4. 使用jre运行JavaFx程序
```shell
#手动编译项目
dir /s /b src\*.java src\*.fxml > sources.txt & javac --module-path %JFX17_PATH%\lib;lib -d mods/javafx @sources.txt & del sources.txt

# 运行, 不再需要 --module-path 指向 JFX17_PATH
jre\bin\java --module-path mods --module com.feng.javafx/com.feng.javafx.HelloApplication
jre\bin\java --module-path mods --m com.feng.javafx/com.feng.javafx.HelloApplication
```

## 构建包含项目模块的jre

1. 编译
```shell
#手动编译项目
dir /s /b src\*.java src\*.fxml > sources.txt & javac --module-path %JFX17_PATH%\lib;lib -d mods/javafx @sources.txt & del sources.txt
```

2. 生成模块文件
```shell
jmod create --class-path mods\javafx com.feng.javafx.jmod
```

3. 利用maven插件将依赖包复制到lib目录
```xml
            <!-- 拷贝依赖的jar包到lib目录 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <!-- ${project.build.directory}是maven变量，内置的，表示target目录,如果不写，将在跟目录下创建/lib -->
                            <outputDirectory>lib</outputDirectory>
                            <!-- excludeTransitive:是否不包含间接依赖包，比如我们依赖A，但是A又依赖了B，我们是否也要把B打进去 默认不打-->
                            <excludeTransitive>false</excludeTransitive>
                            <!-- 复制的jar文件去掉版本信息 -->
                            <stripVersion>true</stripVersion>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

4. 复制当前项目的模块文件到lib目录
```shell
copy /y com.feng.javafx.jmod lib
```

5. 打包jre并生成主类启动器
```shell
jlink --module-path lib --add-modules javafx.controls,javafx.fxml,javafx.web,org.controlsfx.controls,com.dlsc.formsfx,net.synedra.validatorfx,org.kordamp.ikonli.javafx,org.kordamp.bootstrapfx.core,eu.hansolo.tilesfx,com.feng.javafx --output jre --launcher run=com.feng.javafx/com.feng.javafx.HelloApplication
```

6. 运行
```shell
jre\bin\run
```

## 打包为可运行文件
1. 编译
```shell
#手动编译项目
dir /s /b src\*.java src\*.fxml > sources.txt & javac --module-path %JFX17_PATH%\lib;lib -d mods/javafx @sources.txt & del sources.txt
```

2. 生成模块文件
```shell
jmod create --class-path mods\javafx com.feng.javafx.jmod
```

3. 利用maven插件将依赖包复制到lib目录
```xml
            <!-- 拷贝依赖的jar包到lib目录 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy</id>
                        <phase>package</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <!-- ${project.build.directory}是maven变量，内置的，表示target目录,如果不写，将在跟目录下创建/lib -->
                            <outputDirectory>lib</outputDirectory>
                            <!-- excludeTransitive:是否不包含间接依赖包，比如我们依赖A，但是A又依赖了B，我们是否也要把B打进去 默认不打-->
                            <excludeTransitive>false</excludeTransitive>
                            <!-- 复制的jar文件去掉版本信息 -->
                            <stripVersion>true</stripVersion>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
```

4. 复制当前项目的模块文件到lib目录
```shell
copy /y com.feng.javafx.jmod lib
```

5. 生成可执行文件
```shell
## 打包绿色可执行文件
jpackage --type app-image --name javafx --module-path lib --module com.feng.javafx/com.feng.javafx.HelloApplication

## 打包安装exe文件
jpackage --type exe --name javafx --module-path lib --module com.feng.javafx/com.feng.javafx.HelloApplication

## 打包安装msi文件
jpackage --type msi --name javafx --module-path lib --module com.feng.javafx/com.feng.javafx.HelloApplication
```