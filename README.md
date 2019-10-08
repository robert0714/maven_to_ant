# maven_to_ant

## Webapplication

add maven plugin in pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
( omit ...)
  <build>
  <finalName>SampleWeb</finalName>
  <plugins>
   <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-war-plugin</artifactId>
    <version>3.2.3</version>
    <configuration>
     <webResources>
      <resource>
       <directory>src/main/webapp</directory>
       <includes>
        <include>**/*.*</include>
       </includes>
       <targetPath>.</targetPath>
      </resource>
     </webResources>
    </configuration>
   </plugin>
   <plugin>
    <artifactId>maven-resources-plugin</artifactId>
    <version>3.1.0</version>
    <executions>
     <execution>
      <id>copy-resources-1</id>
      <phase>validate</phase>
      <goals>
       <goal>copy-resources</goal>
      </goals>
      <configuration>
       <outputDirectory>${project.build.directory}/ant-project/src</outputDirectory>
       <resources>
        <resource>
         <directory>src/main/java</directory>
         <filtering>true</filtering>
        </resource>
        <resource>
         <directory>src/main/resources</directory>
         <filtering>true</filtering>
        </resource>
       </resources>
      </configuration>
     </execution>
     <execution>
      <id>copy-resources-2</id>
      <phase>validate</phase>
      <goals>
       <goal>copy-resources</goal>
      </goals>
      <configuration>
       <outputDirectory>${project.build.directory}/ant-project/WebContent</outputDirectory>
       <resources>
        <resource>
         <directory>src/main/webapp</directory>
         <filtering>true</filtering>
        </resource>
       </resources>
      </configuration>
     </execution>
     <execution>
      <id>copy-resources-3</id>
      <phase>validate</phase>
      <goals>
       <goal>copy-resources</goal>
      </goals>
      <configuration>
       <outputDirectory>${project.build.directory}/ant-project</outputDirectory>
       <resources>
        <resource>
         <directory>src/ant-template</directory>
         <filtering>true</filtering>
        </resource>
       </resources>
      </configuration>
     </execution>
    </executions>
   </plugin>
   <plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
          <execution>
            <phase>prepare-package</phase>
            <goals>
              <goal>copy-dependencies</goal>
            </goals>
            <configuration>
              <!--http://maven.apache.org/plugins/maven-dependency-plugin/copy-dependencies-mojo.html#includeScope -->
              <includeScope>runtime</includeScope>
              <excludeTransitive>false</excludeTransitive>
              <outputDirectory>${project.build.directory}/ant-project/WebContent/WEB-INF/lib</outputDirectory>
              <overWriteReleases>false</overWriteReleases>
              <overWriteSnapshots>false</overWriteSnapshots>
              <overWriteIfNewer>true</overWriteIfNewer>
            </configuration>
          </execution>
          <execution>
            <id>copy-dependencies-system</id>
            <phase>prepare-package</phase>
            <goals>
              <goal>copy-dependencies</goal>
            </goals>
            <configuration>
              <includeScope>system</includeScope>
              <excludeTransitive>true</excludeTransitive>
              <outputDirectory>${project.build.directory}/ant-project/WebContent/WEB-INF/lib</outputDirectory>
              <overWriteReleases>false</overWriteReleases>
              <overWriteSnapshots>false</overWriteSnapshots>
              <overWriteIfNewer>true</overWriteIfNewer>
            </configuration>
          </execution>
        </executions>
   </plugin>
  </plugins>
 </build>
</project>
```

add ant's build.xml template in {project.basedir}/src/ant-template/build.xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<project basedir="." default="war" name="${project.artifactId}">
 <property name="project.name" value="${project.artifactId}" />
 <property environment="env" />
 <property name="debuglevel" value="source,lines,vars" />
 <property name="target" value="1.8" />
 <property name="source" value="1.8" />

 <property name="lib.dir" value="WebContent/WEB-INF/lib" />
 <path id="Maven Dependencies.libraryclasspath">
  <fileset dir="${lib.dir}">
   <include name="*.jar" />
  </fileset>
 </path>
 <path id="compile.classpath">
  <pathelement location="target/classes" />
  <path refid="Maven Dependencies.libraryclasspath" />
 </path>

 <target name="clean">
    <delete dir="WebContent/WEB-INF/classes" />
  <delete dir="build" />
   <delete dir="dist" />
 </target>

 <target depends="clean" name="cleanall" />

 <target name="init">
  <mkdir dir="build/classes"/>
  <mkdir dir="dist" />
 </target>

 <target depends="init" name="compile"  >
    <echo message="${ant.project.name}: ${ant.file}" />
   <javac encoding="UTF-8" destdir="build/classes"  includeantruntime="false"
    debug="true" srcdir="src">
    <classpath refid="compile.classpath"/>
   </javac>
 </target>

 <target depends="compile" name="build" >
  <copy includeemptydirs="false" todir="build/classes">
   <fileset dir="src">
    <exclude name="**/*.java" />
   </fileset>
  </copy>
 </target>

 <!-- war -->
 <property name="webcontent.dir" value="webapp" />
 <property name="webcontent_class.dir" value="WebContent/WEB-INF/classes" />
 <property name="warpath" value="." />
 <property name="warname" value="${project.name}" />
 <property name="webxmlpath" value="WebContent/WEB-INF/web.xml" />

 <target name="war" depends="cleanall,build">
  <echo message="${ant.project.name}: ${ant.file}" />
  <war destfile="dist/${ant.project.name}.war" webxml="WebContent/WEB-INF/web.xml">
   <fileset dir="WebContent" />
   <classes dir="build/classes"/>
  </war>
 </target>
</project>
```

You can use the below command to generate ant projects

```bash
$ mvn clean package -Dmaven.test.skip=true
$ tree target/ant-project/  -L 1
target/ant-project/
├── build.xml
├── src
└── WebContent
```

You can see the variable "${project.artifactId}" , "${project.name}" changed

we can entry into the folder {project.basedir}/target/ant-project/

```bash
$ cd target/ant-project/
$ ant
```

### Known issues

Test libraies does not need to be packaged into ant's artifect.

# StandAlone Executable jar

add maven plugin in pom.xml

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
( omit ...)
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <spring.version>4.3.23.RELEASE</spring.version>
    <maven.compiler.source>1.8</maven.compiler.source>
    <maven.compiler.target>1.8</maven.compiler.target>
    <main.class>tw.com.xxx.Sample</main.class>
  </properties>
  <build>
  <finalName>SampleJar</finalName>
  <plugins>
    <plugin>
    <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-shade-plugin</artifactId>
        <version>2.4.1</version>
        <executions>
          <execution>
            <id>make-assembly</id>
            <phase>package</phase>
            <goals>
            </goals>
            <configuration>
              <transformer
                implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                <mainClass>com.xxg.Main</mainClass>
              </transformer>
              <transformer
                implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/spring.handlers</resource>
              </transformer>
              <transformer
                implementation="org.apache.maven.plugins.shade.resource.AppendingTransformer">
                <resource>META-INF/spring.schemas</resource>
              </transformer>
            </configuration>
          </execution>
        </executions>
      </plugin>
      <!-- ant_Test start -->
      <plugin>
        <artifactId>maven-resources-plugin</artifactId>
        <version>3.1.0</version>
        <executions>
          <execution>
            <id>copy-resources-1</id>
            <phase>validate</phase>
            <goals>
              <goal>copy-resources</goal>
            </goals>
            <configuration>
              <outputDirectory>${project.build.directory}/ant-project/src</outputDirectory>
              <resources>
                <resource>
                  <directory>src/main/java</directory>
                  <filtering>true</filtering>
                </resource>
                <resource>
                  <directory>src/main/resources</directory>
                  <filtering>true</filtering>
                </resource>
              </resources>
            </configuration>
          </execution>
          <execution>
            <id>copy-resources-2</id>
            <phase>validate</phase>
            <goals>
              <goal>copy-resources</goal>
            </goals>
            <configuration>
              <outputDirectory>${project.build.directory}/ant-project</outputDirectory>
              <resources>
                <resource>
                  <directory>src/ant-template</directory>
                  <filtering>true</filtering>
                </resource>
              </resources>
            </configuration>
          </execution>
        </executions>
      </plugin>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <executions>
          <execution>
            <id>copy-dependencies-runtime</id>
            <phase>prepare-package</phase>
            <goals>
              <goal>copy-dependencies</goal>
            </goals>
            <configuration>
              <!--http://maven.apache.org/plugins/maven-dependency-plugin/copy-dependencies-mojo.html#includeScope -->
              <includeScope>runtime</includeScope>
              <excludeTransitive>false</excludeTransitive>
              <outputDirectory>${project.build.directory}/ant-project/lib</outputDirectory>
              <overWriteReleases>false</overWriteReleases>
              <overWriteSnapshots>false</overWriteSnapshots>
              <overWriteIfNewer>true</overWriteIfNewer>
            </configuration>
          </execution>
          <execution>
            <id>copy-dependencies-system</id>
            <phase>prepare-package</phase>
            <goals>
              <goal>copy-dependencies</goal>
            </goals>
            <configuration>
              <includeScope>system</includeScope>
              <excludeTransitive>true</excludeTransitive>
              <outputDirectory>${project.build.directory}/ant-project/lib</outputDirectory>
              <overWriteReleases>false</overWriteReleases>
              <overWriteSnapshots>false</overWriteSnapshots>
              <overWriteIfNewer>true</overWriteIfNewer>
            </configuration>
          </execution>
        </executions>
      </plugin>
    </plugins>
  </build>
</project>
```

add ant's build.xml template in {project.basedir}/src/ant-template/build.xml

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<project basedir="." default="jar" name="${project.artifactId}">
  <property name="project.name" value="${project.artifactId}" />
  <property environment="env" />
  <property name="debuglevel" value="source,lines,vars" />
  <property name="target" value="1.8" />
  <property name="source" value="1.8" />

  <property name="lib.dir" value="lib" />
  <path id="Maven Dependencies.libraryclasspath">
    <fileset dir="${lib.dir}">
      <include name="*.jar" />
    </fileset>
  </path>
  <path id="compile.classpath">
    <pathelement location="target/classes" />
    <path refid="Maven Dependencies.libraryclasspath" />
  </path>
  
  <target name="clean">
      <delete dir="WebContent/WEB-INF/classes" />
        <delete dir="build" />
        <delete dir="dist" />
  </target>
<!-- clean -->  
  <target depends="clean" name="cleanall" />
<!-- init -->
  <target name="init">
    <mkdir dir="build/classes"/>
    <mkdir dir="dist" />
  </target>

<!-- compile -->
  <target depends="init" name="compile"  >
        <echo message="${ant.project.name}: ${ant.file}" />
      <javac encoding="UTF-8" destdir="build/classes"  includeantruntime="false"
        debug="true" srcdir="src">
        <classpath refid="compile.classpath"/>
      </javac>
  </target>

<!-- build -->  
  <target depends="compile" name="build" >
    <copy includeemptydirs="false" todir="build/classes">
      <fileset dir="src">
        <exclude name="**/*.java" />
      </fileset>
    </copy>
  </target>
  
<!-- jar -->
  <target name="jar" depends="cleanall,build">  
    <pathconvert property="convert.path" pathsep=" ">  
        <path>  
          <!-- lib.dir contains all jar files, in several subdirectories -->  
          <fileset dir="${lib.dir}">
            <include name="*.jar" />  
          </fileset>  
        </path>
        <mapper>  
            <chainedmapper>  
              <!-- remove absolute path -->  
              <flattenmapper />
              <!-- add lib/ prefix -->  
              <globmapper from="*" to="lib/*" />  
            </chainedmapper>  
          </mapper>
        </pathconvert>
    <echo message="${ant.project.name}: ${ant.file}" />  
    <jar destfile="dist/${ant.project.name}.jar" basedir="build/classes" filesetmanifest="skip">
      <zipgroupfileset dir="lib" includes="*.jar" />
      <manifest>
        <attribute name="Main-Class" value="${main.class}" />
        <attribute name="Class-Path" value="${convert.path}" />
      </manifest>
    </jar>
  </target>
</project>
```

You can use the below command to generate ant projects

```bash
$ mvn clean package -Dmaven.test.skip=true
$ tree target/ant-project/  -L 1
target/ant-project/
├── build.xml
├── src
└── lib
```

You can see the variable "${project.artifactId}" , "${project.name}" ,${main.class} changed

we can entry into the folder {project.basedir}/target/ant-project/