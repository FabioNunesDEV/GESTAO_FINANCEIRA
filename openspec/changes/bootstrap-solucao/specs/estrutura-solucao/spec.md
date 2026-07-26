## ADDED Requirements

### Requirement: Solução única com cinco projetos

A solução SHALL se chamar `GEF.sln`, ficar na raiz do repositório e conter exatamente cinco projetos: `GEF.Domain`, `GEF.Application`, `GEF.Infrastructure`, `GEF.UI` e `GEF.Tests`.

#### Scenario: Abertura da solução no Visual Studio

- **WHEN** o arquivo `GEF.sln` é aberto no Visual Studio 2022
- **THEN** o Solution Explorer exibe os cinco projetos e nenhum outro

#### Scenario: Compilação completa

- **WHEN** a solução é compilada em Debug
- **THEN** a compilação termina sem erros

### Requirement: Organização física em src e tests

Os projetos de produção SHALL ficar em `src/` e o projeto de testes em `tests/`, uma pasta por projeto com o mesmo nome do projeto.

#### Scenario: Caminho de cada projeto

- **WHEN** a estrutura de pastas é inspecionada
- **THEN** existem `src/GEF.Domain/GEF.Domain.csproj`, `src/GEF.Application/GEF.Application.csproj`, `src/GEF.Infrastructure/GEF.Infrastructure.csproj`, `src/GEF.UI/GEF.UI.csproj` e `tests/GEF.Tests/GEF.Tests.csproj`

### Requirement: Template e target framework de cada projeto

Cada projeto SHALL usar o template e o target framework abaixo.

| Projeto | Template no Visual Studio | Template dotnet CLI | TFM |
| --- | --- | --- | --- |
| `GEF.Domain` | Class Library | `classlib` | `net8.0` |
| `GEF.Application` | Class Library | `classlib` | `net8.0` |
| `GEF.Infrastructure` | Class Library | `classlib` | `net8.0` |
| `GEF.UI` | WPF Application | `wpf` | `net8.0-windows` |
| `GEF.Tests` | xUnit Test Project | `xunit` | `net8.0` |

#### Scenario: Projeto de UI é WPF para Windows

- **WHEN** `src/GEF.UI/GEF.UI.csproj` é inspecionado
- **THEN** contém `<TargetFramework>net8.0-windows</TargetFramework>`, `<UseWPF>true</UseWPF>` e `<OutputType>WinExe</OutputType>`

#### Scenario: Projetos de biblioteca são multiplataforma

- **WHEN** os `.csproj` de `GEF.Domain`, `GEF.Application` e `GEF.Infrastructure` são inspecionados
- **THEN** cada um contém `<TargetFramework>net8.0</TargetFramework>` e não contém `<UseWPF>`

### Requirement: Regras de dependência entre camadas

As referências entre projetos SHALL seguir exatamente o grafo abaixo, sem nenhuma referência adicional.

| Projeto | Referencia |
| --- | --- |
| `GEF.Domain` | nada |
| `GEF.Application` | `GEF.Domain` |
| `GEF.Infrastructure` | `GEF.Domain`, `GEF.Application` |
| `GEF.UI` | `GEF.Application`, `GEF.Infrastructure` |
| `GEF.Tests` | `GEF.Domain`, `GEF.Application`, `GEF.Infrastructure` |

#### Scenario: Domínio sem dependências

- **WHEN** `src/GEF.Domain/GEF.Domain.csproj` é inspecionado
- **THEN** não contém nenhum elemento `ProjectReference` nem `PackageReference` de biblioteca de infraestrutura

#### Scenario: Aplicação não conhece infraestrutura

- **WHEN** `src/GEF.Application/GEF.Application.csproj` é inspecionado
- **THEN** referencia apenas `GEF.Domain` e não referencia `GEF.Infrastructure` nem `GEF.UI`

#### Scenario: Testes não referenciam a UI

- **WHEN** `tests/GEF.Tests/GEF.Tests.csproj` é inspecionado
- **THEN** não contém referência a `GEF.UI`

#### Scenario: Ausência de ciclo de dependência

- **WHEN** o grafo de referências da solução é percorrido
- **THEN** nenhum projeto é alcançável a partir de si mesmo

### Requirement: Pacotes NuGet por projeto

Os pacotes SHALL ser instalados apenas nos projetos indicados, na versão compatível com .NET 8 (linha `8.x` para pacotes da Microsoft).

| Projeto | Pacotes |
| --- | --- |
| `GEF.Domain` | nenhum |
| `GEF.Application` | nenhum |
| `GEF.Infrastructure` | `Microsoft.EntityFrameworkCore.SqlServer`, `Microsoft.EntityFrameworkCore.Design`, `Microsoft.EntityFrameworkCore.Tools` |
| `GEF.UI` | `Serilog`, `Serilog.Sinks.File`, `Serilog.Sinks.Debug`, `Microsoft.Extensions.DependencyInjection` |
| `GEF.Tests` | `Microsoft.NET.Test.Sdk`, `xunit`, `xunit.runner.visualstudio`, `FluentAssertions` |
| todos (via `Directory.Build.props`) | `StyleCop.Analyzers` |

#### Scenario: EF Core isolado na infraestrutura

- **WHEN** os `.csproj` de `GEF.Domain` e `GEF.Application` são inspecionados
- **THEN** nenhum deles contém `PackageReference` cujo nome comece com `Microsoft.EntityFrameworkCore`

#### Scenario: Versões de EF Core na linha 8

- **WHEN** os `PackageReference` de EF Core em `GEF.Infrastructure` são inspecionados
- **THEN** todas as versões começam com `8.`

#### Scenario: AutoMapper ainda não instalado

- **WHEN** a solução inteira é inspecionada
- **THEN** não existe nenhum `PackageReference` de `AutoMapper`, porque ainda não há DTOs

### Requirement: Propriedades comuns em Directory.Build.props

Um arquivo `Directory.Build.props` SHALL existir na raiz do repositório definindo as propriedades comuns de compilação e o analisador StyleCop para todos os projetos.

#### Scenario: Propriedades aplicadas a todos os projetos

- **WHEN** `Directory.Build.props` é inspecionado
- **THEN** define `Nullable` como `enable`, `ImplicitUsings` como `enable`, `LangVersion` como `latest` e `NeutralLanguage` como `pt-BR`

#### Scenario: StyleCop ativo sem quebrar a compilação

- **WHEN** a solução é compilada com o StyleCop instalado
- **THEN** violações de estilo aparecem como avisos e a compilação não falha

### Requirement: Configuração do StyleCop em stylecop.json

Um arquivo `stylecop.json` SHALL existir na raiz do repositório, ser referenciado como `AdditionalFiles` em `Directory.Build.props` e desligar as regras incompatíveis com as convenções do projeto.

#### Scenario: Arquivo reconhecido pelo analisador

- **WHEN** `Directory.Build.props` é inspecionado
- **THEN** contém um item `AdditionalFiles` apontando para `stylecop.json` com `Link` definido

#### Scenario: Regras contrárias às convenções do projeto desligadas

- **WHEN** `stylecop.json` é inspecionado
- **THEN** as regras SA1600, SA1601, SA1602, SA1101, SA1309, SA1633 e SA1200 estão desabilitadas

#### Scenario: Documentação exigida apenas onde o projeto define

- **WHEN** a solução é compilada
- **THEN** não há aviso SA1600 exigindo documentação de todo membro público

### Requirement: Projeto de teste único

Os testes SHALL ficar em um único projeto `GEF.Tests`, com pastas internas espelhando as camadas testadas.

#### Scenario: Um único projeto de teste na solução

- **WHEN** os projetos da solução são inspecionados
- **THEN** existe exatamente um projeto de teste, chamado `GEF.Tests`

### Requirement: Arquivos do template WPF mantêm o nome original

Os arquivos gerados pelo template WPF SHALL manter os nomes `App.xaml` e `MainWindow.xaml`. A regra de nomenclatura em português se aplica a código de negócio, não a artefatos da plataforma.

#### Scenario: Janela inicial preservada

- **WHEN** `src/GEF.UI` é inspecionado
- **THEN** existem `App.xaml`, `App.xaml.cs`, `MainWindow.xaml` e `MainWindow.xaml.cs`, e o `StartupUri` de `App.xaml` aponta para `MainWindow.xaml`

### Requirement: Teste de fumaça

`GEF.Tests` SHALL conter um teste que passe, provando que a solução compila e o runner de testes está operante.

#### Scenario: Execução do teste de fumaça

- **WHEN** `dotnet test` é executado na raiz da solução
- **THEN** ao menos um teste é encontrado e todos os testes passam

### Requirement: Artefatos de compilação fora do controle de versão

O repositório SHALL ignorar os artefatos de compilação gerados pelos novos projetos.

#### Scenario: Estado limpo após compilar

- **WHEN** a solução é compilada e `git status` é executado
- **THEN** nenhuma pasta `bin`, `obj` ou `.vs` aparece como arquivo não rastreado
