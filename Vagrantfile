# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  config.vm.box = "ferventcoder/win7pro-x64-nocm-lite"

  config.vm.box_check_update = false

  config.vm.communicator = "winrm"
  config.vm.guest = :windows

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  config.vm.provider "virtualbox" do |vb|
     vb.gui = true
     vb.memory = "1024"
  end

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

shell_script = <<-SCRIPT

$installDir = "c:\\Users\\vagrant"
$ftp = "ftp://router-7.bring.out.ba/Main/files/Platform/"
$file = "cygwin.7z"
$file_lo = "libreoffice_core.tar.gz"
$user = "ftpadmin"
$pass = Get-Content( "c:\\vagrant\\ftp_password.config" ) 
Write-Host "Host: $pass"

$destCygwin = Join-Path $installDir $file
$destLO = Join-Path $installDir $file_lo

[System.Console]::Writeline($ftp)
[System.Console]::Writeline($user)
[System.Console]::Writeline($file)
[System.Console]::Writeline($destCygwin)

$webclientFtp = New-Object System.Net.WebClient 
$webclientFtp.Credentials = New-Object System.Net.NetworkCredential($user,$pass)  

<#

Write-host "download $destCygwin"
$uri = New-Object System.Uri($ftp + $file) 
$webclientFtp.DownloadFile($uri, $destCygwin)

Write-host "download $destLO"
$uri_lo = New-Object System.Uri($ftp + $file_lo) 
$webclientFtp.DownloadFile($uri_lo, $destLO )

#>

function Download-File {
param (
  [string]$url,
  [string]$file
 )

  Write-Host "Downloading $url to $file"
  $downloader = new-object System.Net.WebClient
  $downloader.Proxy.Credentials=[System.Net.CredentialCache]::DefaultNetworkCredentials;
  $downloader.DownloadFile($url, $file)
}

# download 7zip
Write-Host "Download 7Zip commandline tool"
$7zaExe = Join-Path $installDir '7za.exe'

# Download-File 'https://chocolatey.org/7za.exe' "$7zaExe"


# unzip the package
# Write-Host "Extracting $file to $destCytwin "
# Start-Process "$7zaExe" -ArgumentList "x -o`"c:\\`" -y `"$destCygwin`"" -Wait -NoNewWindow

# Start-Process "$7zaExe" -ArgumentList "x -o`"c:\\cygwin\\home\\vagrant\\lo`" -y `"$destLO`"" -Wait -NoNewWindow

# iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))

$script = "c:\\cygwin\\home\\vagrant\\script.sh"

$stream = [System.IO.StreamWriter] $script
$stream.Write("#!/bin/bash`n")
$stream.Write("mkdir lo`n")
$stream.Write("echo ``pwd```n")
$stream.Write("cd lo`n")
$stream.Write("tar xvfz /cygdrive/c/Users/vagrant/libreoffice_core.tar.gz`n")
$stream.Write("git checkout -f`n")
$autog = "./autogen.sh --with-junit=$HOME/java/junit-4.10.jar"
$autog = $autog + " --with-ant-home=$HOME/java/apache-ant-1.9.4"
$autog = $autog + " --enable-pch --disable-activex --disable-atl"
$autog = $autog + "  --with-jdk-home=$HOME/java/jdk32"
$autog = $autog + "  --disable-crashdump --disable-gnome-vfs --disable-telepathy"
$autog = $autog + "  --disable-cve-tests --disable-online-update --disable-liblangtag"
$autog = $autog + "  --with-build-version=`"Built by hernad`""
$autog = $autog + "  --without-galleries --with-jpeg-turbo=no"
$autog = $autog + "  --without-doxygen  `n"
$stream.Write( $autog )
$stream.close()

# ConvertTo-LinuxLineEndings( $script ) 
# Get-ChildItem *.sh | ForEach-Object { (Get-Content $_) | Out-File -Encoding UTF8 $_ }

# (Get-Content $script ) | Out-File -Encoding UTF8 $script

$command = @'
$env:Path = "c:\\cygwin\\bin;" + $env:Path
chdir c:\\cygwin\\home\\vagrant
bash script.sh
'@

Invoke-Expression -Command:$command

#Write-Host "shutdown za 2sec"
#shutdown /s /t 2 /c "Reconfiguring vagrant" /f /d p:4:1

# Write-Host "restart za 2sec"
# shutdown /r /t 2 /c "Reconfiguring vagrant" /f /d p:4:1


SCRIPT

  config.vm.provision "shell", inline: shell_script

end
