# escape=`

ARG tag=1803

FROM mcr.microsoft.com/windows/servercore:$tag AS servercore

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ENV MHOLT_ARCHIVER_VERSION 2.0
ENV MHOLT_ARCHIVER_SHA256 c23eab86a8f77e446f2b5dbb03305ac9acfb5187108a0090a4bcb3d908fa9785

RUN [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 ; `
    Invoke-WebRequest $('https://github.com/mholt/archiver/releases/download/v{0}/archiver_windows_amd64.exe' -f $env:MHOLT_ARCHIVER_VERSION) -OutFile archiver.exe -UseBasicParsing ; `
    if ((Get-FileHash archiver.exe -Algorithm sha256).Hash -ne $env:MHOLT_ARCHIVER_SHA256) { Write-Error 'SHA256 mismatch' }

ENV PROMETHEUS_VERSION 2.13.0
ENV PROMETHEUS_SHA256 b30c379c06fc5ff127f0fc2c85010d71a878f207372dbdfe5d6c7af1a38c043f

RUN [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 ; `
    Invoke-WebRequest $('https://github.com/prometheus/prometheus/releases/download/v{0}/prometheus-{0}.windows-amd64.tar.gz' -f $env:PROMETHEUS_VERSION) -OutFile 'prometheus.tar.gz' -UseBasicParsing ; `
    if ((Get-FileHash prometheus.tar.gz -Algorithm sha256).Hash -ne $env:PROMETHEUS_SHA256) { Write-Error 'SHA256 mismatch' }
    
RUN .\archiver.exe open prometheus.tar.gz
RUN Rename-Item -Path $('C:/prometheus-{0}.windows-amd64' -f $env:PROMETHEUS_VERSION) -NewName 'C:/prometheus'

FROM mcr.microsoft.com/windows/nanoserver:$tag

COPY --from=servercore /windows/system32/netapi32.dll /windows/system32/netapi32.dll
COPY --from=servercore /prometheus/                   /prometheus/

EXPOSE 9090

WORKDIR /prometheus
RUN mkdir storage

CMD [ `
    "prometheus.exe", `
    "--config.file=/prometheus/prometheus.yml", `
    "--storage.tsdb.path=/prometheus/storage", `
    "--web.console.libraries=/prometheus/console_libraries", `
    "--web.console.templates=/prometheus/consoles" `
]
