#Delay to spread API Requests
$Random = Get-Random -Maximum 180
Start-Sleep $Random

$StartTime = Get-Date
[Net.ServicePointManager]::SecurityProtocol = "Tls12"
#Endpoint WAN ip
$wanip = invoke-webrequest -Uri icanhazip.com -UseBasicParsing

#Geekflare API
$apikey = "geekflareapikey"
$urlscan = $wanip.content
$Body = @{"url" = $urlscan}
$Header = @{"x-api-key" = $apikey}
$apiendpoint = "https://api.geekflare.com/openport"
$i= Invoke-WebRequest -Uri $apiendpoint -UseBasicParsing -Method POST -Headers $Header -Body ($Body | ConvertTo-Json) -ContentType "application/json"

$APIOutput = $i.content | ConvertFrom-Json
$ConvertTimeStamp = (Get-Date 01.01.1970)+([System.TimeSpan]::fromseconds($APIOutput.timestamp / 1000))
$OpenPorts = $APIOutput.data

if($null -ne $OpenPorts){
    $OpenPorts = $OpenPorts -join ","
}
#Check Documentation For Allow open ports
$AllowedOpenPorts = Ninja-Property-Docs-Get "Firewall" "AllowedOpenPorts"

#Check if there any open ports from API scan
if ($null -eq $OpenPorts){

    Write-Host "No open ports"
    $OpenPorts = "No open ports found"

}else{
    Write-Host "Open Ports Found"

    $AllowedOpenPortsArray = $AllowedOpenPorts.Split(",")
    $OpenPortsArray = $APIOutput.data

    $CurrentStatePorts = foreach ($port in $OpenPortsArray){

        if ($AllowedOpenPortsArray -contains $Port){
            Write-Host "Port:$port Open and Allowed"

        }else{
            $PortResult = "WARNING!:$port Not Allowed to be open"
            $PortResult
        }
    }
}

if ($null -eq $CurrentStatePorts){
    $CurrentStatePorts = "Open Ports Compliant"
}else{
    $CurrentStatePorts = $CurrentStatePorts -join ", "
}


#Create output for customfields
$Output = [pscustomobject][ordered]@{
    Date = $ConvertTimeStamp
    ApiStatus = $APIOutput.apiStatus
    ApiCode = $APIOutput.apiCode
    WANip = $APIOutput.meta.url
    OpenPorts = $OpenPorts
    Compliance = $CurrentStatePorts
    ScanTime = "$((New-Timespan -Start $StartTime -End $(Get-Date)).TotalSeconds) seconds"
}

$Output = $Output | Format-List | Out-String

$Output

Ninja-Property-Set wanOpenPorts $Output
