# SampleDeployIISApi

![C#](https://img.shields.io/badge/C%23-12.0-purple?style=for-the-badge&logo=c-sharp)
![ASP.NET Core](https://img.shields.io/badge/ASP.NET%20Core-8.0-blueviolet?style=for-the-badge&logo=dotnet)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-CI%2FCD-yellow?style=for-the-badge&logo=githubactions)
![IIS](https://img.shields.io/badge/IIS-10.0-green?style=for-the-badge&logo=windows)
![MIT License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge&logo=mit)

## Descrição

SampleDeployIISApi é um projeto de exemplo desenvolvido em ASP.NET Core, criado para demonstrar como configurar um pipeline de CI/CD utilizando GitHub Actions para deploy automatizado em um servidor IIS no Windows. Este projeto é uma referência prática para configurar o GitHub Actions, parando e iniciando o site e o pool de aplicativos do IIS durante o processo de deploy.

## Requisitos

- [.NET 8.0 SDK](https://dotnet.microsoft.com/download/dotnet/8.0)
- [IIS 10.0](https://www.iis.net/downloads)
- [GitHub Runner](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners) configurado como serviço com o usuário `NT AUTHORITY\SYSTEM`

## Instalação

1. Clone o repositório:

    ```sh
    git clone https://github.com/LucasBLs/SampleDeployIISApi.git
    cd SampleDeployIISApi
    ```

2. Configure o GitHub Runner:

    - Siga as instruções da [documentação do GitHub](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners) para configurar o runner.
    - Durante a configuração, escolha rodar o runner como um serviço utilizando o usuário `NT AUTHORITY\SYSTEM`.

## Configuração do GitHub Actions

O arquivo de configuração do GitHub Actions (`.github/workflows/deploy.yml`) deve estar configurado para parar o site e o pool de aplicativos do IIS antes do deploy e iniciá-los após o deploy. Aqui está um exemplo básico:

```yaml
name: Deploy to IIS  # Nome do workflow.

on:
  push:
    branches:
      - main  # Define quando o workflow deve ser acionado (neste caso, em push para o branch main).

jobs:
  build:
    runs-on: self-hosted  # Define o ambiente no qual o trabalho será executado.

    steps:
      - uses: actions/checkout@v2  # Faz o checkout do repositório.

      - name: Restore dependencies
        run: dotnet restore  # Executa dotnet restore para restaurar dependências.

      - name: Build
        run: dotnet build --no-restore  # Executa dotnet build para compilar o projeto sem restaurar as dependências.

      - name: Publish
        run: dotnet publish -c Release -o C:\actions-runner\site  # Executa dotnet publish para publicar o projeto na pasta especificada.

      - name: Stop IIS site and pool
        run: | # Importa o módulo WebAdministration e para o site no IIS.
          Import-Module WebAdministration
          Stop-WebAppPool -Name "SeuPool"
          Stop-WebSite -Name "SeuSite"  

      - name: Deploy
        run: |  # Remove os arquivos antigos e copia os novos arquivos para o diretório do IIS.
          Remove-Item -Recurse -Force C:\inetpub\wwwroot\SeuSite\*
          Copy-Item -Path C:\actions-runner\site\* -Destination C:\inetpub\wwwroot\SeuSite -Recurse 

      - name: Start IIS site and pool
        run: | # Importa o módulo WebAdministration e inicia o site no IIS.
          Import-Module WebAdministration
          Start-WebAppPool -Name "SeuPool"
          Start-WebSite -Name "SeuSite"  
