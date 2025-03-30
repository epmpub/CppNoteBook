### Example of a .NET Standard 2.0 Library

Here's a brief guide on creating a .NET Standard library:

1. **Create a .NET Standard Library:**

   ```
   bash
   Copy code
   dotnet new classlib -n MyLibrary -f netstandard2.0
   ```

2. **Add Code:**

   ```
   csharpCopy codenamespace MyLibrary
   {
       public class Class1
       {
           public static string HelloWorld()
           {
               return "Hello from .NET Standard!";
           }
       }
   }
   ```

3. **Build the Library:**

   ```
   bash
   Copy code
   dotnet build
   ```

4. **Use the DLL in PowerShell 5.1:**

   ```
   powershellCopy code$dllPath = "C:\Path\To\MyLibrary.dll"
   [System.Reflection.Assembly]::LoadFrom($dllPath)
   [MyLibrary.Class1]::HelloWorld()
   ```

By targeting .NET Standard or .NET Framework directly, you can ensure better compatibility with PowerShell 5.1. If you're frequently working with modern .NET Core features, transitioning to PowerShell 7 would be beneficial for a smoother experience.