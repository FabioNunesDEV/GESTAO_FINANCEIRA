## Why

O repositório só contém documentação (`CLAUDE.md`, `README.md`) — não existe solução .NET, nem projetos, nem nenhuma linha de código. Todo trabalho de domínio depende de um esqueleto que respeite as camadas e as regras de dependência já decididas, e criar esse esqueleto sem especificação prévia leva a referências cruzadas erradas que ficam caras de desfazer depois.

## What Changes

- Criação da solução `GEF.sln` com os cinco projetos definidos no `CLAUDE.md`, sob `src/` e `tests/`.
- Definição do template de projeto (tipo de item no Visual Studio) e do TFM de cada projeto.
- Definição das regras de dependência entre camadas, incluindo o que é proibido referenciar.
- Definição de quais pacotes NuGet entram em cada projeto no bootstrap e quais ficam para changes futuras.
- Centralização de propriedades comuns de compilação em `Directory.Build.props`.
- Calibragem do StyleCop em `stylecop.json`, desligando as regras contrárias às convenções do projeto.
- Um teste de fumaça em `GEF.Tests` para provar que a solução compila e o runner de testes funciona.

**Fora de escopo desta change**: entidades de domínio, `DbContext`, migrações, telas funcionais, serviços de aplicação e DTOs. Esta change entrega apenas o esqueleto.

## Forma de execução

Esta change é **executada manualmente pelo usuário no Visual Studio**. Os artefatos servem como especificação e roteiro; o agente não cria arquivos de código. O agente atua em dois momentos:

1. **Mentoria** — orienta passo a passo a criação dos projetos e a adição de cada pacote NuGet.
2. **Validação** — quando o usuário pedir, confere a estrutura criada contra os critérios de aceite deste documento. Estando tudo correto, a change é arquivada.

## Capabilities

### New Capabilities
- `estrutura-solucao`: estrutura física da solução — projetos, camadas, regras de dependência entre elas, TFM, pacotes NuGet por projeto e propriedades comuns de compilação.

### Modified Capabilities
Nenhuma. Não existem specs em `openspec/specs/` — esta é a primeira change do projeto.

## Impact

- **Arquivos criados**: `GEF.sln`, `Directory.Build.props`, `stylecop.json`, cinco arquivos `.csproj` e os arquivos mínimos de cada projeto (`App.xaml`, `App.xaml.cs`, `MainWindow.xaml`, `MainWindow.xaml.cs`, um teste de fumaça).
- **Dependências externas**: pacotes NuGet de EF Core, xUnit, StyleCop e Serilog, conforme detalhado em `design.md`.
- **`.gitignore`**: precisa cobrir artefatos .NET (`bin/`, `obj/`, `.vs/`) — verificar se o atual já cobre.
- **`CLAUDE.md`**: a coluna **Situação** da tabela de bibliotecas passa a `Em uso` para os pacotes efetivamente instalados.
- **Pré-requisitos de ambiente**: .NET 8 SDK, Visual Studio 2022 com a carga de trabalho de desktop .NET, e SQL Server LocalDB instalado (LocalDB só é exercitado em change futura).
