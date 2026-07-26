# Roteiro de execução — Visual Studio 2022

> Execução **manual pelo usuário**. O agente não cria arquivos nesta change.
> Ao final, peça a validação: o agente confere cada cenário de `specs/estrutura-solucao/spec.md` e, estando tudo correto, arquiva a change.
> Caminhos de menu aparecem como `Português (English)`.

## 1. Conferir o ambiente

- [ ] 1.1 Abrir `Ferramentas > Linha de Comando > PowerShell do Desenvolvedor (Tools > Command Line > Developer PowerShell)` e rodar `dotnet --list-sdks`. Confirmar que existe um SDK `8.x`.
- [ ] 1.2 Rodar `dotnet --list-runtimes` e confirmar `Microsoft.WindowsDesktop.App 8.x` — é o runtime que o WPF exige.
- [ ] 1.3 Confirmar no Visual Studio Installer que a carga de trabalho **Desenvolvimento para desktop com .NET (.NET desktop development)** está instalada. Sem ela o template de WPF não aparece.
- [ ] 1.4 Rodar `sqllocaldb info` e confirmar que existe a instância `MSSQLLocalDB`. Só será usada em change futura, mas é melhor descobrir agora se falta instalar.

## 2. Criar a solução na raiz do repositório

O Visual Studio insiste em criar uma subpasta com o nome da solução, o que colocaria o `.sln` em `GESTAO_FINANCEIRA\GEF\`. Como o `.sln` deve ficar na raiz, este único passo sai mais limpo por linha de comando.

- [ ] 2.1 No `PowerShell do Desenvolvedor`, com a pasta atual em `D:\Repositorios\GESTAO_FINANCEIRA`, rodar:
      `dotnet new sln -n GEF`
- [ ] 2.2 Confirmar que `D:\Repositorios\GESTAO_FINANCEIRA\GEF.sln` foi criado.
- [ ] 2.3 Abrir a solução: `Arquivo > Abrir > Projeto/Solução (File > Open > Project/Solution)` e selecionar `GEF.sln`.

> Alternativa só com a IDE: criar uma **Solução em Branco (Blank Solution)** chamada `GEF` dentro de `GESTAO_FINANCEIRA`, fechar o Visual Studio, mover `GEF.sln` uma pasta para cima e apagar a pasta `GEF` que sobrou.

## 3. Criar os cinco projetos

Para cada projeto: clique direito na **solução** no Solution Explorer, `Adicionar > Novo Projeto (Add > New Project)`, escolher o template, e na tela seguinte ajustar **Nome** e **Local (Location)**.

O campo **Local** é o ponto de atenção: o Visual Studio sugere a pasta da solução, e você precisa acrescentar `\src` ou `\tests`. O nome do projeto vira a subpasta automaticamente.

- [ ] 3.1 `GEF.Domain` — template **Biblioteca de Classes (Class Library)**, framework `.NET 8.0`, Local: `D:\Repositorios\GESTAO_FINANCEIRA\src`
- [ ] 3.2 `GEF.Application` — template **Biblioteca de Classes (Class Library)**, framework `.NET 8.0`, Local: `D:\Repositorios\GESTAO_FINANCEIRA\src`
- [ ] 3.3 `GEF.Infrastructure` — template **Biblioteca de Classes (Class Library)**, framework `.NET 8.0`, Local: `D:\Repositorios\GESTAO_FINANCEIRA\src`
- [ ] 3.4 `GEF.UI` — template **Aplicativo WPF (WPF Application)**, framework `.NET 8.0`, Local: `D:\Repositorios\GESTAO_FINANCEIRA\src`
- [ ] 3.5 `GEF.Tests` — template **Projeto de Teste xUnit (xUnit Test Project)**, framework `.NET 8.0`, Local: `D:\Repositorios\GESTAO_FINANCEIRA\tests`
- [ ] 3.6 Apagar o arquivo `Class1.cs` gerado em `GEF.Domain`, `GEF.Application` e `GEF.Infrastructure`.
- [ ] 3.7 Definir `GEF.UI` como projeto de inicialização: clique direito no projeto, `Definir como Projeto de Inicialização (Set as Startup Project)`.
- [ ] 3.8 Em `GEF.UI`, **não renomear** `App.xaml` nem `MainWindow.xaml` — os arquivos do template ficam com o nome original. A regra de português vale para código de negócio; Views e ViewModels novos seguem português (`ContasView`, `ContasViewModel`).
- [ ] 3.9 Em `GEF.Tests`, criar as pastas `Domain`, `Application` e `Infrastructure` (clique direito no projeto > `Adicionar > Nova Pasta`), espelhando as camadas testadas. Projeto de teste é único; a separação é por pasta.

**Cuidados de template:**

- Escolher **WPF Application**, não *WPF Class Library* nem *WPF Application (.NET Framework)*. O `.csproj` correto tem `OutputType=WinExe`, `UseWPF=true` e TFM `net8.0-windows`.
- Escolher **xUnit Test Project**, não MSTest nem NUnit.
- Confirmar que os nomes saíram exatos, com o ponto: `GEF.Domain` e não `GEF_Domain` nem `GEFDomain`.

## 4. Ligar as camadas (referências de projeto)

Clique direito no projeto, `Adicionar > Referência de Projeto (Add > Project Reference)`, marcar os projetos indicados.

- [ ] 4.1 `GEF.Domain` — **nenhuma referência**. Esta camada não conhece ninguém; é o que a mantém testável e estável.
- [ ] 4.2 `GEF.Application` → marcar `GEF.Domain`
- [ ] 4.3 `GEF.Infrastructure` → marcar `GEF.Domain` e `GEF.Application`
- [ ] 4.4 `GEF.UI` → marcar `GEF.Application` e `GEF.Infrastructure`
- [ ] 4.5 `GEF.Tests` → marcar `GEF.Domain`, `GEF.Application` e `GEF.Infrastructure` (**não** marcar `GEF.UI`)
- [ ] 4.6 Compilar a solução (`Ctrl+Shift+B`) e confirmar zero erros.

> Por que a UI referencia a infraestrutura, se isso parece furar a Clean Architecture: a UI é o *composition root* — o lugar onde as implementações concretas são registradas no container. É a única camada autorizada a conhecer a infraestrutura.

## 5. Centralizar propriedades de compilação

- [ ] 5.1 Clique direito na **solução** > `Adicionar > Novo Item (Add > New Item)` > **Arquivo XML (XML File)**, nome `Directory.Build.props`. Confirmar que ele ficou fisicamente ao lado de `GEF.sln`, na raiz.
- [ ] 5.2 Substituir o conteúdo por:

```xml
<Project>

  <PropertyGroup>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
    <LangVersion>latest</LangVersion>
    <NeutralLanguage>pt-BR</NeutralLanguage>
    <EnforceCodeStyleInBuild>true</EnforceCodeStyleInBuild>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="StyleCop.Analyzers" Version="1.1.118">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>

  <ItemGroup>
    <AdditionalFiles Include="$(MSBuildThisFileDirectory)stylecop.json" Link="stylecop.json" />
  </ItemGroup>

</Project>
```

- [ ] 5.3 Clique direito na **solução** > `Adicionar > Novo Item (Add > New Item)` > **Arquivo JSON (JSON File)**, nome `stylecop.json`, também na raiz ao lado de `GEF.sln`. Conteúdo:

```json
{
  "$schema": "https://raw.githubusercontent.com/DotNetAnalyzers/StyleCopAnalyzers/master/StyleCop.Analyzers/StyleCop.Analyzers/Settings/stylecop.schema.json",
  "settings": {
    "documentationRules": {
      "companyName": "Fabio Nunes",
      "documentInterfaces": false,
      "documentExposedElements": false,
      "documentInternalElements": false,
      "documentPrivateElements": false,
      "documentPrivateFields": false,
      "xmlHeader": false
    },
    "orderingRules": {
      "usingDirectivesPlacement": "outsideNamespace",
      "systemUsingDirectivesFirst": true
    },
    "namingRules": {
      "allowCommonHungarianPrefixes": false
    },
    "readabilityRules": {
      "allowBuiltInTypeAliases": false
    }
  }
}
```

- [ ] 5.4 Acrescentar ao `Directory.Build.props`, dentro do `PropertyGroup`, a supressão das regras que continuam disparando mesmo com o `stylecop.json`:

```xml
    <NoWarn>$(NoWarn);SA1101;SA1309;SA1633;SA1600;SA1601;SA1602</NoWarn>
```

- [ ] 5.5 Recompilar. A compilação deve **passar**. Ainda podem sobrar avisos `SA####` de formatação (espaçamento, linhas em branco) — esses são úteis e ficam ligados de propósito. Se aparecer algum SA1600 exigindo documentação, o `stylecop.json` não foi reconhecido: confira o `AdditionalFiles` do passo 5.2.

> **Por que dois lugares.** O `stylecop.json` configura o *comportamento* das regras (`documentExposedElements: false` faz a SA1600 parar de exigir documentação). O `NoWarn` desliga a regra por completo. Regras como SA1101 (`this.` obrigatório) e SA1309 (campo com `_`) não têm chave de configuração no JSON — só saem via `NoWarn`. As de documentação estão nos dois por segurança: se o `AdditionalFiles` falhar, o `NoWarn` ainda segura.

> `Directory.Build.props` é importado automaticamente pelo MSBuild em todo projeto abaixo dele — não precisa referenciar em lugar nenhum. Nunca coloque `TargetFramework` aqui: sobrescreveria o `net8.0-windows` do `GEF.UI` e quebraria o WPF.

> O `AdditionalFiles` usa `$(MSBuildThisFileDirectory)` porque os projetos estão em `src/` e `tests/`, um nível abaixo — caminho relativo simples não acharia o arquivo.

## 6. Instalar os pacotes NuGet

Caminho: clique direito no projeto > `Gerenciar Pacotes NuGet (Manage NuGet Packages)` > aba `Procurar (Browse)` > buscar > escolher a **versão** no painel direito > `Instalar`.

Alternativa mais rápida e menos sujeita a erro de projeto: `Ferramentas > Gerenciador de Pacotes NuGet > Console do Gerenciador de Pacotes (Tools > NuGet Package Manager > Package Manager Console)` e usar `Install-Package`, que aceita o projeto explicitamente.

### 6.1 `GEF.Infrastructure` — Entity Framework Core

- [ ] 6.1.1 Instalar `Microsoft.EntityFrameworkCore.SqlServer`, versão **8.0.x** (a mais recente da linha 8)
- [ ] 6.1.2 Instalar `Microsoft.EntityFrameworkCore.Design`, mesma versão
- [ ] 6.1.3 Instalar `Microsoft.EntityFrameworkCore.Tools`, mesma versão

```powershell
Install-Package Microsoft.EntityFrameworkCore.SqlServer -Version 8.0.11 -ProjectName GEF.Infrastructure
Install-Package Microsoft.EntityFrameworkCore.Design    -Version 8.0.11 -ProjectName GEF.Infrastructure
Install-Package Microsoft.EntityFrameworkCore.Tools     -Version 8.0.11 -ProjectName GEF.Infrastructure
```

**Mentoria:**

- **Não aceite a versão 9.x ou superior que o NuGet oferece por padrão.** O projeto é `net8.0`; EF Core 9 exige .NET 9. Filtre pela linha `8.0.*`. Ajuste o `8.0.11` acima para a última `8.0.*` disponível — o número exato não importa, a linha importa.
- Os três pacotes têm papéis diferentes: `SqlServer` é o provider de runtime; `Design` habilita o *scaffolding* de migrações; `Tools` traz os cmdlets `Add-Migration` e `Update-Database` para o Package Manager Console.
- `Design` e `Tools` são ferramentas de desenvolvimento. O Visual Studio já os instala com `PrivateAssets="all"`, o que impede que vazem para quem referenciar `GEF.Infrastructure`. Confira no `.csproj` — se não estiver lá, acrescente.
- Instale **apenas neste projeto**. Se EF Core aparecer em `GEF.Domain` ou `GEF.Application`, a separação de camadas está perdida — é o erro mais comum neste tipo de bootstrap.

### 6.2 `GEF.UI` — Serilog e container de DI

- [ ] 6.2.1 Instalar `Serilog`
- [ ] 6.2.2 Instalar `Serilog.Sinks.File`
- [ ] 6.2.3 Instalar `Serilog.Sinks.Debug`
- [ ] 6.2.4 Instalar `Microsoft.Extensions.DependencyInjection`, versão **8.0.x**

```powershell
Install-Package Serilog                                  -ProjectName GEF.UI
Install-Package Serilog.Sinks.File                       -ProjectName GEF.UI
Install-Package Serilog.Sinks.Debug                      -ProjectName GEF.UI
Install-Package Microsoft.Extensions.DependencyInjection -Version 8.0.1 -ProjectName GEF.UI
```

**Mentoria:**

- Serilog e o container ficam na UI porque configurar log e registrar dependências é **inicialização de aplicação**, e isso pertence ao composition root. Nenhuma outra camada instala Serilog.
- Quando as camadas internas precisarem logar, elas vão receber `ILogger<T>` (de `Microsoft.Extensions.Logging.Abstractions`), não Serilog. Assim trocar a biblioteca de log um dia não toca em domínio.
- Os pacotes `Serilog.*` não têm amarra de versão com o .NET — pegar a estável mais recente está correto. Já os `Microsoft.Extensions.*` seguem a linha do runtime: use `8.0.*`.
- Os dois sinks entram nesta change, mas a configuração deles (caminho `%LOCALAPPDATA%\GEF\logs\gef-.log`, retenção de 30 dias, nível por ambiente) fica para a change de inicialização da UI — está especificada no `CLAUDE.md`.
- Nada será logado ainda: sem código chamando, pacote instalado não tem efeito em runtime.

### 6.3 `GEF.Tests` — asserções

- [ ] 6.3.1 Conferir que o template já trouxe `Microsoft.NET.Test.Sdk`, `xunit`, `xunit.runner.visualstudio` e `coverlet.collector`. Não reinstale.
- [ ] 6.3.2 Instalar `FluentAssertions`, versão **7.x**

```powershell
Install-Package FluentAssertions -Version 7.0.0 -ProjectName GEF.Tests
```

**Mentoria:**

- **Atenção à licença:** a partir da versão 8 o FluentAssertions passou a ter licença comercial paga. A linha 7.x continua sob Apache 2.0. O NuGet vai oferecer a 8.x por padrão — recuse e fixe em `7.x`. Se preferir não depender disso, o `Assert` do xUnit resolve; perde-se legibilidade nas mensagens de falha.
- Os três pacotes de teste que já vêm no template têm papéis distintos: `Microsoft.NET.Test.Sdk` é a infraestrutura de execução, `xunit` é o framework e `xunit.runner.visualstudio` é o que faz os testes aparecerem no Test Explorer. Faltando o runner, `dotnet test` acha zero testes.

### 6.4 O que **não** instalar agora

- [ ] 6.4.1 Confirmar que nenhum destes foi instalado: `AutoMapper`, `Swashbuckle.AspNetCore`, `SendGrid`, `Hangfire`.

**Mentoria:** todos estão previstos no `CLAUDE.md`, nenhum tem uso nesta fase. AutoMapper sem DTO não mapeia nada; Swagger sem API não documenta nada; SendGrid sem notificação e Hangfire sem job agendado só somam dependência para manter atualizada. Cada um entra na change que criar a necessidade real.

## 7. Teste de fumaça

- [ ] 7.1 Renomear `UnitTest1.cs` para `TesteFumacaTests.cs` em `GEF.Tests` (aceitar a renomeação da classe que o Visual Studio oferece).
- [ ] 7.2 Substituir o conteúdo por:

```csharp
using FluentAssertions;
using Xunit;

namespace GEF.Tests;

/// <summary>
/// Prova que a solução compila e o runner de testes está operante.
/// </summary>
public class TesteFumacaTests
{
    [Fact]
    public void SolucaoCompilaEExecutaTestes()
    {
        var resultado = 2 + 2;

        resultado.Should().Be(4);
    }
}
```

- [ ] 7.3 Abrir `Testar > Gerenciador de Testes (Test > Test Explorer)`, rodar tudo e confirmar 1 teste verde.
- [ ] 7.4 No terminal, rodar `dotnet test` na raiz e confirmar que passa também fora da IDE.

## 8. Verificação final e commit

- [ ] 8.1 `Compilar > Recompilar Solução (Build > Rebuild Solution)` — zero erros.
- [ ] 8.2 Rodar `git status` e confirmar que nenhuma pasta `bin`, `obj` ou `.vs` aparece como não rastreada.
- [ ] 8.3 Confirmar a árvore final:

```
GESTAO_FINANCEIRA/
├── GEF.sln
├── Directory.Build.props
├── stylecop.json
├── src/
│   ├── GEF.Domain/GEF.Domain.csproj
│   ├── GEF.Application/GEF.Application.csproj
│   ├── GEF.Infrastructure/GEF.Infrastructure.csproj
│   └── GEF.UI/GEF.UI.csproj
└── tests/
    └── GEF.Tests/GEF.Tests.csproj
```

- [ ] 8.4 Pedir ao agente a **validação da change**: ele confere cada cenário de `specs/estrutura-solucao/spec.md` contra o que foi criado e aponta divergências.
- [ ] 8.5 Depois da validação, atualizar no `CLAUDE.md` a coluna **Situação** para `Em uso` nas linhas de EF Core, StyleCop e Serilog.
- [ ] 8.6 Commitar e arquivar a change.
