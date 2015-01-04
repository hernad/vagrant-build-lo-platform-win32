# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  #config.vm.box = "ferventcoder/win7pro-x64-nocm-lite"
  config.vm.box = "w7-choco-vs2013community"

  config.vm.box_check_update = false

  config.vm.communicator = "winrm"
  config.vm.guest = :windows

  # config.vm.network "private_network", ip: "192.168.33.10"

  config.vm.provider "virtualbox" do |vb|
     vb.gui = true
     vb.memory = "2048"
  end

shell_script = <<-SCRIPT

$installDir = "c:\\Users\\vagrant"
$ftp = "ftp://router-7.bring.out.ba/Main/files/Platform/"
$file = "cygwin-3.7z"
$file_lo = "libreoffice_core.tar.gz"
$file_java = "home_java.tar.gz"
$user = "ftpadmin"
Write-Host "U vagrant direktoriju se nalazi ftp_password.config"
$pass = Get-Content( "c:\\vagrant\\ftp_password.config" ) 
Write-Host "ftp pasword je: $pass"

$destCygwin = Join-Path $installDir $file
$destLO = Join-Path $installDir $file_lo
$destJava = Join-Path $installDir $file_java

[System.Console]::Writeline($ftp)
[System.Console]::Writeline($user)
[System.Console]::Writeline($file)
[System.Console]::Writeline($destCygwin)

$webclientFtp = New-Object System.Net.WebClient 
$webclientFtp.Credentials = New-Object System.Net.NetworkCredential($user,$pass)  


$uri = New-Object System.Uri($ftp + $file) 
[System.Console]::Writeline( "download: " + $uri)
$webclientFtp.DownloadFile($uri, $destCygwin)

$uri_lo = New-Object System.Uri($ftp + $file_lo) 
[System.Console]::Writeline( "download: " + $uri_lo)
$webclientFtp.DownloadFile($uri_lo, $destLO)

$uri_java = New-Object System.Uri($ftp + $file_java) 
[System.Console]::Writeline( "download: " + $uri_java)
$webclientFtp.DownloadFile($uri_java, $destJava)


function Download-File {
param (
  [string]$url,
  [string]$file
 )

  [System.Console]::Writeline("Downloading $url to $file")

  $downloader = new-object System.Net.WebClient
  $downloader.Proxy.Credentials=[System.Net.CredentialCache]::DefaultNetworkCredentials;
  $downloader.DownloadFile($url, $file)
}

# download 7zip
[System.Console]::Writeline( "Download 7za from chocolatey")
$7zaExe = Join-Path $installDir '7za.exe'

Download-File 'https://chocolatey.org/7za.exe' "$7zaExe"


# unzip the package
[System.Console]::Writeline( "Extract cygwin to " + $destCygwin)
Start-Process "$7zaExe" -ArgumentList "x -o`"c:\\`" -y `"$destCygwin`"" -Wait -NoNewWindow

$script = "c:\\cygwin\\home\\vagrant\\script.sh"

[System.Console]::Writeline( "Generisem script za pokretanje LO build-a" )

$stream = [System.IO.StreamWriter] $script
$stream.Write("#!/bin/bash`n")
$stream.Write("tar xvfz /cygdrive/c/Users/vagrant/$file_java`n")
$stream.Write("mkdir lo`n")
$stream.Write("echo ``pwd```n")
$stream.Write("cd lo`n")
$stream.Write("tar xvfz /cygdrive/c/Users/vagrant/libreoffice_core.tar.gz`n")
$stream.Write("git checkout -f`n")
$stream.Write("export PATH=/opt/lo/bin:/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin:`$PATH`n")
$autog = "./autogen.sh --with-junit=`$HOME/java/junit-4.10.jar"
$autog = $autog + " --with-ant-home=`$HOME/java/apache-ant-1.9.4"
$autog = $autog + " --enable-pch "
# --disable-activex --disable-atl
$autog = $autog + "  --with-jdk-home=`$HOME/java/jdk"
$autog = $autog + "  --disable-crashdump --disable-gnome-vfs --disable-telepathy"
$autog = $autog + "  --disable-cve-tests --disable-online-update --disable-liblangtag"
$autog = $autog + "  --with-build-version=`"Built by hernad`""
$autog = $autog + "  --without-galleries --with-jpeg-turbo=no"
$autog = $autog + "  --without-doxygen  `n"
$stream.Write( $autog )
$stream.Write("/opt/lo/bin/make`n")
$stream.close()

$command = @'
$env:Path = "c:\\cygwin\\bin;" + $env:Path
chdir c:\\cygwin\\home\\vagrant
bash script.sh
'@

Invoke-Expression -Command:$command

#Write-Host "shutdown za 2sec"
#shutdown /s /t 2 /c "Reconfiguring vagrant" /f /d p:4:1

SCRIPT

  config.vm.provision "shell", inline: shell_script

end
