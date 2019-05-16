# escape=`

ARG tag=1809

FROM mcr.microsoft.com/windows/servercore:$tag AS servercore

SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

ENV MHOLT_ARCHIVER_VERSION 2.0
ENV MHOLT_ARCHIVER_SHA256 c23eab86a8f77e446f2b5dbb03305ac9acfb5187108a0090a4bcb3d908fa9785

RUN Invoke-WebRequest $('https://github.com/mholt/archiver/releases/download/v{0}/archiver_windows_amd64.exe' -f $env:MHOLT_ARCHIVER_VERSION) -OutFile archiver.exe -UseBasicParsing ; `
    if ((Get-FileHash archiver.exe -Algorithm sha256).Hash -ne $env:MHOLT_ARCHIVER_SHA256) { Write-Error 'SHA256 mismatch' }

ENV PROMETHEUS_VERSION 2.9.2

RUN Invoke-WebRequest $('https://github.com/prometheus/prometheus/releases/download/v{0}/prometheus-{0}.windows-amd64.tar.gz' -f $env:PROMETHEUS_VERSION) -OutFile 'prometheus.tar.gz' -UseBasicParsing
RUN .\archiver.exe open prometheus.tar.gz
RUN Rename-Item -Path $('C:/prometheus-{0}.windows-amd64' -f $env:PROMETHEUS_VERSION) -NewName 'C:/download'

FROM mcr.microsoft.com/windows/nanoserver:$tag

COPY --from=servercore /windows/system32/netapi32.dll /windows/system32/netapi32.dll
COPY --from=servercore /download/prometheus.exe      /bin/prometheus.exe
COPY --from=servercore /download/promtool.exe        /bin/promtool.exe
COPY --from=servercore /download/prometheus.yml      /etc/prometheus/prometheus.yml
COPY --from=servercore /download/console_libraries/  /etc/prometheus/
COPY --from=servercore /download/consoles/           /etc/prometheus/

RUN mkdir prometheus

EXPOSE 9090

WORKDIR /bin

CMD [ `
    "prometheus.exe", `
    "--config.file=/etc/prometheus/prometheus.yml", `
    "--storage.tsdb.path=/prometheus", `
    "--web.console.libraries=/etc/prometheus/console_libraries", `
    "--web.console.templates=/etc/prometheus/consoles" `
]
