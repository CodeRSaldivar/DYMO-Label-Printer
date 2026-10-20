#!ps
#maxlength=50000000
#timeout=3600000

Push-Location -Path C:\Users\Public


Invoke-WebRequest 'https://us-cloud-storage.s3.us-east-2.amazonaws.com/Dymo-Drivers.zip' -OutFile Dymo-Drivers.zip

Expand-Archive -Path '.\Dymo-Drivers.zip' -DestinationPath '.\' -Force

Remove-Item -Path '.\Dymo-Drivers.zip' -Force -ea 0


pnputil /add-driver .\Dymo-Drivers\*.inf /subdirs /install


Invoke-WebRequest 'https://us-cloud-storage.s3.us-east-2.amazonaws.com/Dymo-Templates.zip' -OutFile Dymo-Templates.zip

Expand-Archive -Path '.\Dymo-Templates.zip' -DestinationPath '.\' -Force

Remove-Item -Path '.\Dymo-Templates.zip' -Force -ea 0

Write-Host `n"Copying Label Files..."

Copy-Item -Path ".\Dymo-Templates\*.*" -Destination "C:\Program Files\Common Files\eClinicalWorks\plugin\Resources\DymoTemplates\" -Force

"https://dymoreleasecontent.blob.core.windows.net/dymo-release/DLS8Setup8.7.4.exe" | Out-File -FilePath ".\Dymo-Installation-Notes.txt"

"https://dymoreleasecontent.blob.core.windows.net/dymo-release/DCDWIN/DCDSetup1.6.1.7-X64.exe" | Out-File -FilePath ".\Dymo-Installation-Notes.txt" -Append

"C:\Program Files\Common Files\eClinicalWorks\plugin\Resources\DymoTemplates\Specimen.dymo" | Out-File -FilePath ".\Dymo-Installation-Notes.txt" -Append

"C:\Program Files\Common Files\eClinicalWorks\plugin\Resources\DymoTemplates\Custom550_final.dymo" | Out-File -FilePath ".\Dymo-Installation-Notes.txt" -Append

"https://chromewebstore.google.com/detail/printerlogic-extension-v1/bfgjjammlemhdcocpejaompfoojnjjfn" | Out-File -FilePath ".\Printerlogic-Extension-URL.txt" -Append


try {
    $response = Invoke-WebRequest 'https://dymoreleasecontent.blob.core.windows.net/dymo-release/DCDWIN/DCDSetup1.6.1.7-X64.exe' -OutFile DCDSetup1.6.1.7-X64.exe -ErrorAction Stop

    Write-Host `n"Installing DYMO Connect 1.6.1.7"`n

    $processes = "msiexec"

    foreach ($process in $processes) {

        Get-Process -Name "$process" -ea 0 | Stop-Process -Force -ea 0

    }

    $process = Start-Process -FilePath '.\DCDSetup1.6.1.7-X64.exe' -ArgumentList '/S /v"/qn /norestart"' -Verb RunAs -PassThru -Wait

    if ($process.ExitCode -eq 0) {

        Remove-Item -Path '.\DCDSetup1.6.1.7-X64.exe' -Force -ea 0

        Write-Host "Process finished successfully!" -ForegroundColor Green

    } else {

        Write-Warning "Process failed with exit code: $($process.ExitCode)" -ForegroundColor Red

    }

    Remove-Item -Path 'C:\Users\Public\Desktop\DYMO Label v.8.lnk' -Force -ea 0

} 

catch {

    Write-Error "Download failed: $_"

    if ($_.Exception.Response) {

        Write-Host "HTTP Status Code: $($_.Exception.Response.StatusCode.value__)"

    }

}


Pop-Location
