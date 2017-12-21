---
layout: post
title:  代码管理中的静态资源引用
category: tech
---

项目经常引用一些本地的jar包，有些由于年代久远或种种原因，在中央仓库和私服中都找不到。
在项目管理中，ant和maven都遇到了这个问题:
# ant
由于ant编译过程中需要自己编写build脚本，因此只要将外部jar包放在固定位置，在打包时加入指定文件夹中即可（如WEB-INF/lib）

编译过程中需要将lib加入到classpath中
```
<path id="classpath">
    <fileset dir="${basedir}/lib">
        <include name="*.jar"/>
    </fileset>
    <fileset dir="${ivy.lib.dir}">
        <include name="*.jar"/>
    </fileset>
</path>

<target name="compile" depends="init, ivy-retrieve">
    <javac includeantruntime="false" encoding="utf-8" target="1.6" debuglevel="lines,vars,source" debug="on"
    srcdir="${src.dir}" destdir="${build.class.dir}" optimize="true" verbose="false">
        <classpath refid="classpath"></classpath>
    </javac>
</target>
```

打包过程中不要忘记copy
```
<target name="build" depends="compile">
     <copy todir="${build.web}/lib">
         <fileset dir="${basedir}/lib">
             <include name="*.jar"/>
         </fileset>
     </copy>
 </target>
```


# maven
在maven中就比较系统化了
引用本地jar包的方式和引用远程仓库的方式相同，这种设计让人很舒服，指定的路径也可以看做为本地仓库的一部分。其中这里，关键内容是systemPath，system;其他的随便写写就好，因为此时已经不需要使用groupId和artifactId来对资源进行定位了。
```
<dependency>
    <groupId>xxx</groupId>
    <artifactId>xxx</artifactId>
    <version>0.0.1</version>
    <scope>system</scope>
    <systemPath>${project.basedir}/lib/qiyemail_validator_pg.jar</systemPath>
</dependency>
```
scope为system的引用只是在编译过程中起作用，而并不会打包进指定位置，这里还是需要在<build>内手动指定资源的路径，如下：
```
<resources>
    <resource>
        <directory>src/main/resources</directory>
    </resource>
    <resource>
        <directory>lib</directory>
        <targetPath>BOOT-INF/lib/</targetPath>
        <includes>
            <include>**/*.jar</include>
        </includes>
    </resource>
</resources>
```
