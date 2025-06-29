Add necessary packages to test projects:

```powershell
# Assume tests folder is $pwd
ls -Filter *.csproj -Recurse -Name | % { 
    dotnet package add GitHubActionsTestLogger --project $_ 
    dotnet package add Microsoft.Testing.Extensions.CrashDump --project $_ 
    dotnet package add Microsoft.Testing.Extensions.HangDump --project $_
    dotnet package add Microsoft.Testing.Extensions.CodeCoverage --project $_ 
  }
```
