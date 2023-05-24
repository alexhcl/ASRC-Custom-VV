# AppScan Source (SAST) and AzureDevOps Integration
</br>
It will help to Integrate AppScan Source on AzureDevOps. It will enable AzureDevOps to start scan, generate report, publish results to AppScan Source Database and AppScan Enterprise and check for Security Gate.<br>
<br>
Requirements:<br>
1 - AppScan Source in Windows Server.<br>
2 - Add AppScan Source bin folder to Windows PATH Environment Variable.<br>
3 - Install Azure Pipeline Agent for Windows in same Windows Server that has AppScan Source. https://learn.microsoft.com/en-us/azure/devops/pipelines/agents/windows-agent?view=azure-devops<br>
3.1 - Add Azure Pipeline Agentr as a Service.<br>
3.2 - Change User Service to same User that has access in AppScan Enterprise.<br>
4 - Create AppScan Enterprise token <install_dir>\bin\ounceautod.exe -u username -p password --persist.<br>
  Source: https://help.hcltechsw.com/appscan/Source/10.0.8/topics/ounce_auto_login.html <br>
  <br>

```yaml
variables:
  aseApiKeyId: xxxxxxxxxxxxxxxx
  aseApiKeySecret: xxxxxxxxxxxxxxxx
  compiledArtifactFolder: none
  scanConfig: Normal scan
  aseAppName: $(Build.Repository.Name)
  aseHostname: xxxxxxxxxxxxxxxx
  aseToken: C:\ProgramData\HCL\AppScanSource\config\ounceautod.token
  sevSecGw: highIssues
  maxIssuesAllowed: 100
  WorkingDirectory: $(System.DefaultWorkingDirectory)
  BuildNumber: $(Build.BuildNumber)

trigger: none

pool: default

steps:
- pwsh: |
    Invoke-WebRequest -Uri https://raw.githubusercontent.com/jrocia/AppScan-onprem-and-AzureDevOps-integration/main/scripts/appscanase_create_application_ase.ps1 -OutFile appscanase_create_application_ase.ps1
    .\appscanase_create_application_ase.ps1
  displayName: 'Checking Application ID in ASE'
- pwsh: |
    Invoke-WebRequest -Uri https://raw.githubusercontent.com/jrocia/AppScan-onprem-and-AzureDevOps-integration/main/scripts/appscansrc_create_config_scan_folder.ps1 -OutFile appscansrc_create_config_scan_folder.ps1
    .\appscansrc_create_config_scan_folder.ps1
  displayName: 'Creating AppScan Source Config file'
- pwsh: |
    Invoke-WebRequest -Uri https://raw.githubusercontent.com/jrocia/AppScan-onprem-and-AzureDevOps-integration/main/scripts/appscansrc_scan.ps1 -OutFile appscansrc_scan.ps1
    .\appscansrc_scan.ps1
  displayName: 'Running SAST scan'
- pwsh: |
    Invoke-WebRequest -Uri https://raw.githubusercontent.com/jrocia/AppScan-onprem-and-AzureDevOps-integration/main/scripts/appscansrc_publish_assessment_to_enterprise.ps1 -OutFile appscansrc_publish_assessment_to_enterprise.ps1
    .\appscansrc_publish_assessment_to_enterprise.ps1
  displayName: 'Publishing Result Scan into ASE'
- pwsh: |
    Invoke-WebRequest -Uri https://raw.githubusercontent.com/jrocia/AppScan-onprem-and-AzureDevOps-integration/main/scripts/appscansrc_check_security_gate.ps1 -OutFile appscansrc_check_security_gate.ps1
    .\appscansrc_check_security_gate.ps1
  displayName: 'Checking Security Gate'
- publish: $(aseAppName)-$(BuildNumber).pdf
  artifact: $(aseAppName)-$(BuildNumber).pdf
  continueOnError: On
  displayName: 'Uploading PDF to Azure Artifacts'
```
