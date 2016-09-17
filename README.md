This is a [Vagrant](https://www.vagrantup.com/) Environment for a [SonarQube](http://www.sonarqube.org) based Source Code Analysis service.

This will:

* Install a SonarQube instance and configure it through its [Web API](http://docs.sonarqube.org/display/DEV/Web+API).
* Install [Visual Studio Community 2015](https://www.visualstudio.com/en-us/products/visual-studio-community-vs.aspx).
* Install the [MSBuild SonarQube Runner](http://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+MSBuild).
* Analyse a C# project ([MailBounceDetector](https://github.com/rgl/MailBounceDetector)); including its [XUnit](https://xunit.github.io/) Unit Tests and an [OpenCover](https://github.com/OpenCover/opencover) Code Coverage report.


# Usage

Build the base box with:

```bash
git clone https://github.com/joefitzgerald/packer-windows
cd packer-windows
# this will take ages so leave it running over night...
packer build windows_2012_r2.json
vagrant box add windows_2012_r2 windows_2012_r2_virtualbox.box
rm *.box
cd ..
```

Install the needed plugins:

```bash
vagrant plugin install vagrant-reload # https://github.com/aidanns/vagrant-reload 
```

Then start this environment:

```bash
vagrant up
```


# Excluding Projects from SonarQube Analysis

[`sonar.exclusions`](http://docs.sonarqube.org/display/SONAR/Narrowing+the+Focus)
(and other paths) are relative to the project directory, as such, they can only
exclude files inside a project. to exclude the project itself you have to use
one of the following (before calling `MSBuild.SonarQube.Runner begin`):

1. Modify each `.csproj` file to have the following element:

    ```xml
    <PropertyGroup>
      <SonarQubeExclude>true</SonarQubeExclude>
    </PropertyGroup>
    ```

1. Create a `<Project>.csproj.user` file (on the same directory as `<Project>.csproj`) with:

    ```xml
    <Project xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
      <PropertyGroup>
        <SonarQubeExclude>true</SonarQubeExclude>
      </PropertyGroup>
    </Project>
    ```

1. Implement some global logic by placing a MSBuild snippet (e.g. a `.targets` file)
   inside the current user or the local machine MSBuild directories, that is, at one of:

    ```
    C:\Users\<User>\AppData\Local\Microsoft\MSBuild\14.0\Microsoft.Common.targets\ImportBefore
    C:\Program Files (x86)\MSBuild\14.0\Microsoft.Common.Targets\ImportBefore
    ```

   **NB** these directories are scanned before the `<Project>.csproj.user` file.

   **NB** before importing the files MSBuild sorts them by file name; e.g. to import a file
   last giveit a, e.g. `ZZZ` prefix.

For more information see:

* [SonarQube jira issue (closed as won't fix): Add the ability to filter the projects to be analyzed by path](https://jira.sonarsource.com/browse/SONARMSBRU-191)
* [SonarQube for Visual Studio Plugin: Appendix 3: Advanced SonarQube Scanner for MSBuild configuration](https://github.com/SonarSource-VisualStudio/sonar-.net-documentation/blob/master/doc/appendix-3.md)
* [SonarQube: Excluding Artifacts from the Analysis](http://docs.sonarqube.org/display/SCAN/Excluding+Artifacts+from+the+Analysis)
* [SonarQube: Narrowing the Focus](http://docs.sonarqube.org/display/SONAR/Narrowing+the+Focus#NarrowingtheFocus-patterns)
* [SonarQube: Analyzing with SonarQube Scanner for MSBuild](http://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+MSBuild)
* [MSBuild Documentation](https://msdn.microsoft.com/en-us/library/dd393574.aspx)