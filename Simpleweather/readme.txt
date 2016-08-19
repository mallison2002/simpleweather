SIMPLE WEATHER EXAMPLE CMD-LINE NON ARTIFACTORY - LOCAL PTX SERVER
==================================================================================================

# change settings to local repo 
cd ~/.m2
ln -s settings.xml.protexlocalrepo settings.xml

# remove artifacts in local repo
rm -r /CORAL/working_repository/maven/maven_local_repo/repo/*
rm -r ~/.m2/repository/*

# compile
cd /CORAL/working_repository/maven/maven_local_repo/applications/simpleweather_statestreet/
mvn install
#mvn exec:java -Dexec.mainClass=org.sonatype.mavenbook.weather.Main -Dexec.args="90024"
java  -Djava.compiler=NONE -cp target/simple-weather-1.0-jar-with-dependencies.jar org.sonatype.mavenbook.weather.Main 90024

# show downloads directly from maven2 central
# show that the repo has lots of JARs, bit no approvals or management 
ls /CORAL/working_repository/maven/maven_local_repo/repo/

# clean
rm -r ~/.m2/repository/*
rm -r /CORAL/working_repository/maven/maven_local_repo/repo/*
cd ~/.m2
rm settings.xml
ln -s settings.xml.artifactory settings.xml

Artifactory, login & clean all repos
Protex delete 12_SimpleWeather_StateStreet
CodeCenter delete 12_SimpleWeather_StateStreet and create empty


	 
==================================================================================================

SIMPLE WEATHER SETUP 

cd /CORAL/working_repository/maven/maven_local_repo/applications

# DONE
# create the archetype in dir/ simple-weather
#mvn archetype:generate -DgroupId=org.sonatype.mavenbook.custom       -DartifactId=simple-weather       -Dpackage=org.sonatype.mavenbook       -Dversion=1.0
#> choose 251

# DONE
# clear null data
#cd src
#rm ./main/java/org/sonatype/mavenbook/App.java ./test/java/org/sonatype/mavenbook/AppTest.java

# DONE
# add all java src files from here :
#http://www.sonatype.com/books/mvnex-book/reference/customizing-sect-simple-weather-source.html#ex-simple-weather-model-object



# START HERE
# NOTE make sure /home/mallison/.m2/settings.xml has :
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <localRepository>/CORAL/working_repository/maven/maven_local_repo/repo</localRepository>
  <interactiveMode>true</interactiveMode>
  <usePluginRegistry>false</usePluginRegistry>
  <offline>false</offline>
</settings>

# compile
cd ./simpleweather_statestreet
mvn install

# Run
mvn exec:java -Dexec.mainClass=org.sonatype.mavenbook.weather.Main -Dexec.args="90024"

# List all project dependencies
mvn dependency:resolve
mvn dependency:tree
mvn dependency:list 

##### OPTION TO DOWNLOAD DEPS LOCALLY (EG FOR SACN)
# download deps locally  (they are in target/dependency)
# add to POM to allow download of dependencies locally
       <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-dependency-plugin</artifactId>
        <version>2.8</version>
        <executions>
          <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
              <goal>copy-dependencies</goal>
            </goals>
            <configuration>
              <outputDirectory>
	      	/CORAL/code_repository/simpleweather/dependencies
	      </outputDirectory>
              <overWriteReleases>false</overWriteReleases>
              <overWriteSnapshots>true</overWriteSnapshots>
              <excludeTransitive>true</excludeTransitive>
            </configuration>
          </execution>
        </executions>
      </plugin>

mvn dependency:copy-dependencies

# run tests
mvn test

###### OPTION TO Package to JAR
# add to the pom to enable pulling deps in
# http://books.sonatype.com/mvnex-book/reference/customizing-sect-custom-packaged.html

            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>


# create exe JAR
mvn install assembly:assembly
 
# run 
java -cp target/simple-weather-1.0-jar-with-dependencies.jar org.sonatype.mavenbook.weather.Main 90024
 
 
# clean
mvn clean:clean


##################################################################
# PROTEX PLUGIN 
# add protex stuff to the POM in the maven repo
  <pluginRepositories>
    <pluginRepository>
      <id>protex</id>
      <name>Black Duck Protex Plugins</name>
      <url>http://localhost.localdomain:8087/repo/maven2</url>
    </pluginRepository>


     <plugin>
        <groupId>com.blackducksoftware.plugins</groupId>
        <artifactId>protex-maven-plugin</artifactId>
        <version>1.0</version>
        <configuration>
          <username>mallison@blackducksoftware.com</username>
          <password>valis81</password>
        </configuration>
      </plugin>

# reporting plugin

     <plugin>
        <groupId>com.blackducksoftware.plugins</groupId>
        <artifactId>protex-maven-plugin</artifactId>
        <version>1.0</version>
        <reportSets>
          <reportSet>
            <reports>
              <report>summary</report>
            </reports>
          </reportSet>
        </reportSets>
      </plugin>
    </plugins>


# add protex stuff to the settings.xml in ~.m2

  <localRepository>/CORAL/working_repository/maven/repo</localRepository>

   <pluginGroups>
    <pluginGroup>com.blackducksoftware.plugins</pluginGroup>
  </pluginGroups>
  
  
# create protex project 
mvn  protex:create-project -DfailOnExistingProject=false

# run analysis
mvn protex:analyze

# reports
# summaries in target/site dir
mvn protex:summary
mvn protex:bill-of-materials















##################################################################
# JENKINS

#  install plugins:

ARTIFACTORY PLUGIN
#################################

get from:
http://updates.jenkins-ci.org/download/plugins/artifactory/2.1.5 <<<<<<<<<<<<<<<<< IMPORTANT MAYBE !!
I used jenkins 1.5.14 <<<<<<<<<<<<<<<<< IMPORTANT MAYBE !!

plugin manager > advanced > Upload plugin


PROTEX PLUGIN
#################################

/CORAL/working_repository/jenkins/download_plugins/Protex-Jenkins-Plugin_v0.2/
plugin manager > advanced > Upload plugin

FILESYSTEM PLUGIN
#################################

File System SCM
plugin manager > advanced > Upload plugin


PORTS
#################################
# make sure to run jenkins:8083
in /etc/init.d/jenkins add httpPort=8083 
... daemon --user "$JENKINS_USER" --pidfile "$JENKINS_PID_FILE" $JAVA_CMD $PARAMS --httpPort=8083 > /dev/null

or :::

JENKINS_PORT="8083"
JENKINS_JAVA_OPTIONS="-Xmx2048m -XX:MaxPermSize=512m"

...JAVA_CMD="$JENKINS_JAVA_CMD $JENKINS_JAVA_OPTIONS -DJENKINS_HOME=$JENKINS_HOME -jar $JENKINS_WAR"


Configure Jenkins
#################################

Jenkins > Manage Jenkins > Config system

Home directory	/var/lib/jenkins

Maven Project Configuration
Global MAVEN_OPTS
Local Maven Repository default ~/.m2


Jenkins Location
Jenkins URL http://localhost:8083/

Artifactory
Artifactory servers http://192.168.1.100:8081/artifactory
Default Deployer Credentials
admin
password

########################################################################################
# create Jenkins project
Create maven 2/3 project: '12_SimpleWeather_StateStreet' 
- (make this the same name as CC application)

Maven project name:	12_SimpleWeather_StateStreet

Source Code Management
===========================

x File System
Path: /CORAL/working_repository/maven/maven_local_repo/applications/simpleweather_statestreet/src/

Build Triggers: 
===========================
x Build whenever a SNAPSHOT dependency is built


Build
===========================
Root POM pom.xml

Post Steps: Protex create and scan
===========================
Protex Server URL: http://192.168.1.100:8087
Protex User Name: mallison@blackducksoftware.com
Protex Password:
Protex Project Name: 12_SimpleWeather_StateStreet
Create Protex Project? x
Scan Protex Project? x
Protex Project Source Path: $workspace

Build Environment: 
===========================
x Resolve artifacts from Artifactory 
artifactory server http://192.168.1.100:8081/artifactory/
resolution -- To use Artifactory for resolution select a virtual repository --
x override default resolution credentials
username admin
pass password

Post-build Actions:
===========================
Deploy artifacts from Artifactory 
artifactory server http://192.168.1.100:8081/artifactory/
Target releases repository ext-release-local
Target snapshots repository ext-snapshot-local
x Capture and publish build info

x Run Black Duck Code Center compliance checks (requires Artifactory Pro)
Code Center application name: 12_SimpleWeather_StateStreet
Code Center application version: 1
x Auto-create missing component requests
x Auto-discard stale component requests


########################################################################################
## Code Center 

Create Application
Name 12_SimpleWeather_StateStreet
Version 1
Workflow FaststartStandard
User mallison
Role application admin

########################################################################################
## run Jenkins build

- show artifactory jcenter-cache repo with deps
- show artifactory ext-release-local deployed simpleweather JAR
- show artifactory build : governance for 12_SimpleWeather_StateStreet
- show protex scan
- show CC BOM : this gets populated from with the dependencies:
	Apache Log4j	1.2.14	
	Apache ORO	2.0.8	
	Apache Velocity	1.5	
	International Components for Unicode - ICU4J	2.6.1	
	JDOM	1.0	
	JUnit	3.8.1	
	Lang	2.1	
	XML Commons External Components XML APIs	1.0.b2		
	XOM	1.0	
	commons-collections	3.1	
	dom4j	1.6.1	
	jaxen	1.1.1	
	xalan	2.6.0	
	xercesImpl	2.6.2
	xmlParserAPIs	2.6.2

## jenkins
- creates build-info.json in 02-Simpleweather-Artifactory/target/ directory
- this is used by BDS plugin to populate CC BOM ????

## Protex
- Jenkins protex plugin allows auto scan of src code
- dependencies are not included ??

########################################################################################
## PTX IDENTIFICATION

# Id the src files
Main, Weather etc


########################################################################################
## run CC validation
## CC artifacts vs PTX src code

CC Admin > Settings Protex
- Add Protex Server
- Name Local Protex
Host Addr: http://192.168.1.100:8087
Username mallison@blackducksoftware.com
Pass :val*

CC mallison > Protex Credentials
- create protex credential
- Username mallison@blackducksoftware.com
- Pass :val*
- local protex

Applications > Validate
Server: Centos Protex
Application: 12-Simpleweather-StateStreet
Associate

Validate
( if sync problem, Admin > Application Setting > Sync Now )

Show new comps in Code Center only
- Create Componnets in  Protex
- Show new comps in Protex now as KB items


 

########################################################################################
################################### END #####################################################
########################################################################################
########################################################################################










=================================================================================================

MAVEN GUIDE

# put settings.xml file in ~mallison/.m2
# point to default repo in /CORAL/working_repository/maven_local_repo

# setup maven project: my-app in /CORAL/working_repository/maven_local_repo/applications

cd  /CORAL/working_repository/maven_local_repo/applications
mvn archetype:generate \
  -DarchetypeGroupId=org.apache.maven.archetypes \
  -DgroupId=com.mycompany.app \
  -DartifactId=my-app

# clean compile  
cd my-app
mvn clean

# java compile
cd my-app
mvn compile

# compile, test, create jar
mvn package

# test
mvn test

# run
mvn exec:java -Dexec.mainClass=org.sonatype.mavenbook.weather.Main -Dexec.args="90024"

=================================================================================================

SIMPLE WEATHER EXAMPLE SALESDEMO

cd /CORAL/working_repository/maven/maven_local_repo/applications

# create the archetype in dir/ simple-weather
mvn archetype:generate -DgroupId=org.sonatype.mavenbook.custom       -DartifactId=simple-weather       -Dpackage=org.sonatype.mavenbook       -Dversion=1.0
> choose 251

# clear null data
cd src
rm ./main/java/org/sonatype/mavenbook/App.java ./test/java/org/sonatype/mavenbook/AppTest.java

# add all java src files from here :
http://www.sonatype.com/books/mvnex-book/reference/customizing-sect-simple-weather-source.html#ex-simple-weather-model-object

# compile
cd ./simpleweather_salesdemo
mvn install

# Run
mvn exec:java -Dexec.mainClass=org.sonatype.mavenbook.weather.Main -Dexec.args="94401"

# List all project dependencies
mvn dependency:resolve
 
# add protex stuff to the POM in the maven repo

    <pluginRepository>
      <id>protex</id>
      <name>Black Duck Protex Plugins</name>
      <url>http://localhost/repo/maven2</url>
    </pluginRepository>
    
          <plugin>
        <groupId>com.blackducksoftware.plugins</groupId>
        <artifactId>protex-maven-plugin</artifactId>
        <version>1.0</version>
        <configuration>
          <username>mallison@blackducksoftware.com</username>
          <password>valis81</password>
        </configuration>
      </plugin>

# reporting plugin

     <plugin>
        <groupId>com.blackducksoftware.plugins</groupId>
        <artifactId>protex-maven-plugin</artifactId>
        <version>1.0</version>
        <reportSets>
          <reportSet>
            <reports>
              <report>summary</report>
            </reports>
          </reportSet>
        </reportSets>
      </plugin>
    </plugins>


# add protex stuff to the settings.xml in ~.m2

  <localRepository>/CORAL/working_repository/maven/repo</localRepository>

   <pluginGroups>
    <pluginGroup>com.blackducksoftware.plugins</pluginGroup>
  </pluginGroups>
  
  
# create protex project 
mvn  protex:create-project -DfailOnExistingProject=false

# run analysis
mvn protex:analyze

# reports
# summaries in target/site dir
mvn protex:summary
mvn protex:bill-of-materials

=======================================================================

cd  /CORAL/working_repository/maven/applications

mvn archetype:generate -DgroupId=org.sonatype.mavenbook.simple \
-DartifactId=simple \
-Dpackage=org.sonatype.mavenbook \
-Dversion=1.0-SNAPSHOT


  
=======================================================================

cd  /CORAL/working_repository/maven/applications

mvn archetype:create -DgroupId=com.test -DartifactId=mytest 
  

########################################################################################
# import code into Subversion

SIMPLEWEATHER

cd /CORAL/working_repository/subversion/

# done
# create
svnadmin create --fs-type fsfs  /CORAL/working_repository/artifactory/subversion/repo

svn ls file:///CORAL/working_repository/artifactory/repo

#import filesys
svn import /CORAL/working_repository/artifactory/simpleweather file:///CORAL/working_repository/artifactory/repo/simpleweather -m 'lite Import'
#svn import /CORAL/working_repository/subversion/local_workspace/protex_std_demo_lite_workspace file:///CORAL/working_repository/artifactory/repo/protex_std_demo_lite_workspace/ -m 'lite Import'
#svn import /CORAL/working_repository/subversion/local_workspace/j-ftp file:///CORAL/working_repository/artifactory/repo/j-ftp -m 'lite Import'

# check out to filesys for editing
mkdir local_workspace
cd local_workspace
svn checkout file:///CORAL/working_repository/artifactory/repo/simpleweather

# get info on the checked out code
cd simpleweather
svn info

# delete data frmom local copy, check changes back in and check the repo
svn delete <file or dir>
svn commit
svn ls file:///CORAL/working_repository/artifactory/repo/simpleweather/target
