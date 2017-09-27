Although there is a beta version of the powershell available, the PowerCLI only seems to work with alpha release. If not using PowerCLI beta releases are recommended.

The installation method for RHEL systems are little different but the following steps should work on CentOS7 and OEL7.

**Installing Powershell**

    #) sudo yum -y install https://github.com/PowerShell/PowerShell/releases/download/v6.0.0-alpha.18/powershell-6.0.0_alpha.18-1.el7.centos.x86_64.rpm
    
- This installs powershell in /opt/microsoft/powershell/6.0.0-alpha.18/ directory.

- You can verify if powershell is installed by simply typing powershell on the command line and it will take you ta a powershell prompt as follows:
       
            [root@localhost ~]# powershell
            PowerShell
            Copyright (C) Microsoft Corporation. All rights reserved.

            PS /root>
                
- Verify that the modules are loaded properly by issuing any powershell command like such:

            PS /root> Write-Host "This is Powershell"
            This is Powershell

**Insalling PowerCLI Core**
    
- Download the PowerCLI zip file
    
            #) curl -sSL -o PowerCLI_Core.zip https://download3.vmware.com/software/vmw-tools/powerclicore/PowerCLI_Core.zip
    
- Move the PowerCLI zip file to Powershell Modules directory located in /opt/microsoft/powershell/6.0.0-alpha.18/Modules/ and unzip it as follows.
        
            #) mv PowerCLI_Core.zip //opt/microsoft/powershell/6.0.0-alpha.18/Modules/
            #) cd /opt/microsoft/powershell/6.0.0-alpha.18/Modules/
            #) unzip PowerCLI_Core.zip
        
- Unzipping creates the following files
    
        __MACOSX  open_source_license.txt  PowerCLI.Cis  PowerCLI.Cis.zip  PowerCLI_Core.zip  PowerCLI.Vds.zip  PowerCLI.ViCore.zip  README.md  Start-PowerCLI.ps1
        
- Unzip the PowerCLI.Cis.zip, PowerCLI_Core.zip and PowerCLI.Vds.zip in the same directory. PowerCLI can function without other files and only these three modules are 
needed but its wiser to keep all of them as they are.

            #) unzip PowerCLI.Cis.zip
            #) unzip PowerCLI_Core.zip
            #) unzip PowerCLI.Vds.zip
    
    
**Test PowerCLI**
    
- To test PowerCLI is installed properly do the following:
    
            #) powershell
             PS /root> Get-Module -ListAvailable PowerCLI* | Import-Module
        
- Now try connecting to vcenter:
        
            PS /root> Connect-VIServer -Server vcenter.server.com   -User DOMAIN\username   -Password "YourPassword"
        
NOTE:
    
        - You are likely to see an error like below
    
            Error
            Connect-VIServer : 9/26/17 03:15:36 PM Connect-VIServer The libcurl library in use (7.29.0) and its SSL backend (“NSS/3.21 Basic ECC”) do not support custom 
            handling of certificates. A libcurl built with OpenSSL is required.
            At line:1 char:1
            + Connect-VIServer -Server vcenter.example.com -User administ …
            + ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
            + CategoryInfo : NotSpecified: (:) [Connect-VIServer], ViError
            + FullyQualifiedErrorId : Client20_ConnectivityServiceImpl_Reconnect_Exception,VMware.VimAutomation.ViCore.Cmdlets.Commands.ConnectVIServer
            
    
- This error is because CentOS 7 default curl is built with NSS/3.21, while, .NET needs libcurl compiled with OpenSSL. In order to resolve the issue, curl built in
with CentOS 7 can’t be used and must be replaced.        
        
    - Do the following to resolve this:
            
            #) yum -y groupinstall “Development Tools”
            #) curl -sSL -o curl-7.52.1.tar.gz https://curl.haxx.se/download/curl-7.52.1.tar.gz
            #) tar -zxvf curl-7.52.1.tar.gz
            #) cd curl-7.52.1
            #) ./configure
            #) make
            #) make install
        
- Verify that the curl version is built with OpenSSL
            
            #) /usr/local/bin/curl --version
                curl 7.52.1 (x86_64-pc-linux-gnu) libcurl/7.52.1 OpenSSL/1.0.1e
                
                
- Modify LD_CONFIG_PATH

    LD_LIBRARY_PATH is an environment variable that can be used to provide list of directories, when a program needs to search for shared libraries. The libcurl library 
    was installed in /usr/local/lib, when curl was compiled and installed from source.

    - Follow the procedure below to set the path appropriately.
            
            #) cd /etc/ld.so.conf.d/
            
        - Create a file called libcurl.conf and enter
                
                /usr/loca/lib
                 
        in it
                
    - Run "ldconfig" to update the shared library cache.
            
            #) ldconfig
            
**User PowerCLI to connect to vCenter**
    
        #) powershell
        PS /root> Get-Module -ListAvailable PowerCLI* | Import-Module
        PS /root> Connect-VIServer -Server vcenter.server.com -User DOMAIN\username -Password "YourPassword"

- If you get an invalid certificate warning set powercli configuration to ignore as such:
    
        #) powershell
        PS /root> Get-Module -ListAvailable PowerCLI* | Import-Module
        PS /root> Connect-VIServer -Server vcenter.server.com -User DOMAIN\username -Password "YourPassword"
        PS /root> Set-PowerCLIConfiguration -InvalidCertificateAction ignore -confirm:$false
    
    
**Example scripts**

- List snapshots for a vm
        
    vim test.ps1
        
        # First import all the core modules for powercli and then disable certificate warning error and set confirm to false so that no user input is necessary for automation.
        Import-Module -Name PowerCLI.Cis
        Import-Module -Name PowerCLI.Vds
        Import-Module -Name PowerCLI.ViCore
        Set-PowerCLIConfiguration -InvalidCertificateAction ignore -confirm:$false
        Connect-VIServer -Server vcenter.server.com -User DOMAIN\username -Password "YourPassword"
    
        Get-VM -name vm_name | Get-Snapshot
    
    
    
    
    
