# Tutorial: Install and use packages with MSBuild in Visual Studio

- 09/26/2024

Terminal options

This tutorial shows you how to create a C++ "Hello World" program that uses the `fmt` library with MSBuild, vcpkg, and Visual Studio. You'll install dependencies, configure the project, build, and run a simple application.



## Prerequisites

- [Visual Studio](https://visualstudio.microsoft.com/downloads/) with C++ development workload
- [Git](https://git-scm.com/downloads)
- Windows 7 or newer



## 1 - Set up vcpkg

1. Clone the repository

   The first step is to clone the vcpkg repository from GitHub. The repository contains scripts to acquire the vcpkg executable and a registry of curated open-source libraries maintained by the vcpkg community. To do this, run:

   Console

   ```console
   git clone https://github.com/microsoft/vcpkg.git
   ```

   The vcpkg curated registry is a set of over 2,000 open-source libraries. These libraries have been validated by vcpkg's continuous integration pipelines to work together. While the vcpkg repository does not contain the source code for these libraries, it holds recipes and metadata to build and install them in your system.

2. Run the bootstrap script

   Now that you have cloned the vcpkg repository, navigate to the `vcpkg` directory and execute the bootstrap script:

   Console

   ```console
   cd vcpkg; .\bootstrap-vcpkg.bat
   ```

   The bootstrap script performs prerequisite checks and downloads the vcpkg executable.

   That's it! vcpkg is set up and ready to use.

1. Integrate with Visual Studio MSBuild

   The next step is to enable user-wide vcpkg integration, this makes MSBuild aware of vcpkg's installation path.

   Run

   Console

   ```console
   .\vcpkg.exe integrate install
   ```

   This outputs:

   Console

   ```console
   All MSBuild C++ projects can now #include any installed libraries. Linking will be handled automatically. Installing new libraries will make them instantly available.
   ```



## 2 - Set up the Visual Studio project

1. Create the Visual Studio project

   - Create a new project in Visual Studio using the "Console Application" template

     ![create a new C++ Windows console application](https://learn.microsoft.com/en-us/vcpkg/resources/get_started/visual-studio-create-project-msbuild.png)

     Screenshot of the Visual Studio UI for showing how to create a new C++ Windows console application in Visual Studio

     

   - Name your project "helloworld"

   - Check the box for "Place solution and project in the same directory."

   - Click the "Create" button

     ![naming your MSBuild C++ project](https://learn.microsoft.com/en-us/vcpkg/resources/get_started/visual-studio-name-project-msbuild.png)

     Screenshot of Visual Studio UI for naming your MSBuild C++ project and clicking the "create" button.

     

2. Configure the `VCPKG_ROOT` environment variable.

    Note

   Setting environment variables in this manner only affects the current terminal session. To make these changes permanent across all sessions, set them through the Windows System Environment Variables panel.

   Open the built-in Developer PowerShell window in Visual Studio.

   ![opening built-in developer powershell](https://learn.microsoft.com/en-us/vcpkg/resources/get_started/visual-studio-developer-powershell.png)

   Screenshot of Visual Studio UI for the built-in PowerShell developer window

   

   Run the following commands:

   PowerShell

   ```PowerShell
   $env:VCPKG_ROOT = "C:\path\to\vcpkg"
   $env:PATH = "$env:VCPKG_ROOT;$env:PATH"
   ```

   ![setting up your environment variables](https://learn.microsoft.com/en-us/vcpkg/resources/get_started/visual-studio-environment-variable-setup-powershell.png)

   Screenshot of Visual Studio UI for the built-in PowerShell developer window showing how to set up VCPKG_ROOT and and add it to PATH.

   

   Setting `VCPKG_ROOT` helps Visual Studio locate your vcpkg instance. Adding it to `PATH` ensures you can run vcpkg commands directly from the shell.

3. Generate a manifest file and add dependencies.

   Run the following command to create a vcpkg manifest file (`vcpkg.json`):

   Console

   ```console
   vcpkg new --application
   ```

   The [`vcpkg new`](https://learn.microsoft.com/en-us/vcpkg/commands/new) command adds a `vcpkg.json` file and a `vcpkg-configuration.json` file in the project's directory.

   Add the `fmt` package as a dependency:

   Console

   ```console
   vcpkg add port fmt
   ```

   Your `vcpkg.json` should now contain:

   JSON

   ```json
   {
       "dependencies": [
           "fmt"
       ]
   }
   ```

   This is your manifest file. vcpkg reads the manifest file to learn what dependencies to install and integrates with MSBuild to provide the dependencies required by your project.

   The generated `vcpkg-configuration.json` file introduces a [baseline](https://learn.microsoft.com/en-us/vcpkg/reference/vcpkg-configuration-json#registry-baseline) that places [minimum version constraints](https://learn.microsoft.com/en-us/vcpkg/users/versioning) on the project's dependencies. Modifying this file is beyond the scope of this tutorial. While not applicable in this tutorial, it's a good practice to keep the `vcpkg-configuration.json` file under source control to ensure version consistency across different development environments.



## 3 - Set up the project files

1. Modify the `helloworld.cpp` file.

   Replace the content of `helloworld.cpp` with the following code:

   C++

   ```cpp
   #include <fmt/core.h>
   
   int main()
   {
       fmt::print("Hello World!\n");
       return 0;
   }
   ```

   This source file includes the `<fmt/core.h>` header which is part of the `fmt` library. The `main()` function calls `fmt::print()` to output the "Hello World!" message to the console.

    Note

   The code editor may underline the lines referencing `fmt` files and symbols as errors. You need to build your project once for vcpkg to install the dependencies and make auto-completion tools evaluate the code correctly.



## 4 - Enable manifest mode

1. Navigate to your Project Properties page.

   Using the menu navigation at the top, choose **Project > Properties**. A new window will open.

2. Navigate to **Configuration Properties > vcpkg**, and set `Use vcpkg Manifest` to `Yes`.

   ![Enable manifest mode in project properties](https://learn.microsoft.com/en-us/vcpkg/resources/get_started/visual-studio-manifest-msbuild.png)

   Screenshot of enabling vcpkg manifest mode in Visual Studio Project Properties

   

   Other settings, such as [triplets](https://learn.microsoft.com/en-us/vcpkg/users/triplets), are filled in with default values vcpkg detects from your project and will be useful when configuring your project.



## 5 - Build and run the project

1. Build the project.

   Build the project using the `Build > Build Solution` option from the top menu.

   If MSBuild detects a `vcpkg.json` file and manifests are enabled in your project, MSBuild installs the manifest's dependencies as a pre-build step. Dependencies are installed in a `vcpkg_installed` directory in the project's build output directory. Any headers installed by the library can be directly used, and any libraries installed will be automatically linked.

2. Run the application.

   Finally, run the executable:

   ![Running the executable](https://learn.microsoft.com/en-us/vcpkg/resources/get_started/visual-studio-msbuild-project.png)

   Screenshot of Visual Studio UI for running the executable.

   

   You should see the output:

   ![Program output](https://learn.microsoft.com/en-us/vcpkg/resources/get_started/visual-studio-msbuild-output.png)

   Screenshot of the program outputs - "Hello World!"

   