## Context

O repositório tem apenas documentação. As decisões de arquitetura já estão registradas no `CLAUDE.md` (WPF + MVVM, Clean Architecture simples, EF Core, LocalDB, xUnit, cinco projetos com prefixo `GEF`), mas nada foi materializado em código.

Restrições que moldam este design:

- **Execução manual**: o usuário cria a estrutura no Visual Studio 2022; o agente especifica e depois valida. Por isso o roteiro precisa ser executável por menus da IDE, não só por CLI.
- **Projeto pequeno e de uso pessoal**: complexidade adicional só entra quando houver necessidade concreta.
- **Windows 11 + .NET 8**: `GEF.UI` é WPF, logo fica preso ao TFM `net8.0-windows`. As demais camadas não devem herdar essa amarra.

## Goals / Non-Goals

**Goals:**

- Esqueleto compilável com as cinco camadas e o grafo de dependências correto.
- Regra de dependência explícita e verificável, para que erro de referência seja detectado na validação e não meses depois.
- Pacotes NuGet instalados apenas onde pertencem, com EF Core confinado à infraestrutura.
- Propriedades de compilação centralizadas, evitando repetição nos cinco `.csproj`.
- Um teste passando, provando que a cadeia compilar → testar funciona ponta a ponta.

**Non-Goals:**

- Entidades de domínio, `DbContext`, migrações e connection string.
- Telas funcionais, navegação, injeção de dependência configurada, classe base de ViewModel.
- DTOs e mapeamento (por isso AutoMapper fica fora).
- Swagger, SendGrid e Hangfire — sem API, sem e-mail e sem job nesta fase.
- CI, publicação e instalador.

## Decisions

### 1. `src/` e `tests/` em vez de projetos na raiz

Convenção dominante no ecossistema .NET, mantém a raiz legível e separa código de produção de código de teste. Custo: um nível extra de caminho.

**Alternativa considerada**: projetos como pastas irmãs na raiz. Descartada porque com `openspec/`, `.historico/`, `.claude/` e mais cinco pastas de projeto a raiz fica difícil de ler.

### 2. `Directory.Build.props` na raiz para propriedades comuns

MSBuild importa esse arquivo automaticamente em todo projeto abaixo dele. `Nullable`, `ImplicitUsings`, `LangVersion` e o `PackageReference` do StyleCop ficam em um lugar só. Atende a regra "não repetir código" do `CLAUDE.md` no nível de build.

**Alternativa considerada**: repetir as propriedades nos cinco `.csproj`. Descartada — cinco pontos de divergência.

**Alternativa considerada**: Central Package Management (`Directory.Packages.props` com `ManagePackageVersionsCentrally`). Boa prática, mas adiciona indireção que atrapalha quem está aprendendo a estrutura agora. Fica como evolução quando o número de pacotes crescer.

### 3. StyleCop como aviso, nunca como erro, com `stylecop.json` desde o início

`TreatWarningsAsErrors` fica desligado. Em projeto recém-criado o StyleCop dispara centenas de avisos (SA1600 de documentação, ordem de usings, etc.); transformar isso em erro travaria o desenvolvimento no primeiro dia.

Um `stylecop.json` na raiz entra **nesta change**, calibrando as regras antes do primeiro código: evita que a lista de avisos nasça poluída e que regras contrárias às decisões do projeto (documentação obrigatória em tudo, `this.` obrigatório) passem a impressão de que o código está errado quando está seguindo o `CLAUDE.md`.

Regras desligadas e o motivo:

| Regra | Motivo |
| --- | --- |
| SA1600, SA1601, SA1602 | O `CLAUDE.md` pede summary em classes e métodos importantes, não em todo membro |
| SA1101 (`this.` obrigatório) | Contraria o estilo padrão do C# moderno |
| SA1309 (campo não pode iniciar com `_`) | `_campoPrivado` é a convenção do próprio .NET |
| SA1633 (cabeçalho de licença) | Projeto pessoal, sem cabeçalho em arquivo |
| SA1200 (usings dentro do namespace) | Mantido o padrão de usings no topo do arquivo |

### 4. EF Core confinado a `GEF.Infrastructure`

`GEF.Domain` e `GEF.Application` ficam sem pacote nenhum. É o que sustenta a Clean Architecture: se `Application` puder referenciar EF Core, em algum momento um `DbContext` vaza para dentro de um caso de uso e a camada perde o sentido.

Os três pacotes de EF Core têm papéis distintos:

| Pacote | Para que serve |
| --- | --- |
| `Microsoft.EntityFrameworkCore.SqlServer` | Provider do SQL Server/LocalDB — o runtime propriamente dito |
| `Microsoft.EntityFrameworkCore.Design` | Suporte de tempo de projeto; necessário para `Add-Migration` funcionar |
| `Microsoft.EntityFrameworkCore.Tools` | Cmdlets do Package Manager Console (`Add-Migration`, `Update-Database`) |

`Design` e `Tools` são ferramentas de desenvolvimento, não deveriam ir para produção — daí `PrivateAssets="all"`, que o Visual Studio já aplica sozinho ao instalar.

### 5. `Microsoft.Extensions.DependencyInjection` em `GEF.UI`

Mesmo sem DI configurada nesta change, o pacote entra porque o container é responsabilidade da camada mais externa (composition root). Deixá-lo instalado evita que, na primeira necessidade, o container seja registrado no lugar errado.

**Alternativa considerada**: instalar depois. Aceitável, mas o pacote é pequeno e a decisão de onde ele mora é justamente o que se quer fixar agora.

### 6. Serilog em `GEF.UI`

A configuração do logger é inicialização de aplicação, portanto pertence ao composition root. As camadas internas, quando precisarem logar, receberão uma abstração (`ILogger<T>` de `Microsoft.Extensions.Logging`), nunca o Serilog direto — isso mantém a troca de biblioteca de log possível sem tocar em domínio.

### 7. FluentAssertions em `GEF.Tests`

Torna a asserção legível (`resultado.Should().Be(10)`) e as mensagens de falha muito mais informativas que `Assert.Equal`. Não é obrigatório, mas em cálculo financeiro — onde a maioria das asserções é sobre valores e coleções — o ganho de diagnóstico é real.

**Atenção de licença**: FluentAssertions mudou para licença paga a partir da versão 8. A versão 7.x continua sob Apache 2.0. Fixar em `7.x` para uso gratuito.

### 8. `NeutralLanguage` como `pt-BR`

Coerente com a decisão de código de negócio em português. Evita que mensagens de recurso assumam inglês por padrão.

## Risks / Trade-offs

- **StyleCop gera avalanche de avisos no início** → mantido como aviso, e `stylecop.json` disponível para calibrar as regras ruidosas.
- **`net8.0-windows` em `GEF.UI` amarra a UI ao Windows** → aceito: o `CLAUDE.md` define WPF em Windows 11 como alvo. As camadas internas ficam em `net8.0` puro, então uma UI alternativa no futuro não exige reescrever domínio.
- **Pacotes instalados sem uso imediato (Serilog, DI)** → risco baixo: nenhum efeito em runtime enquanto não houver código chamando. O benefício é fixar a camada onde cada um pertence.
- **FluentAssertions 8 tem licença comercial** → fixar em 7.x e registrar a restrição aqui; a alternativa é usar só `Assert` do xUnit.
- **Criação manual pode divergir da spec** → mitigado pela etapa de validação: cada cenário do `spec.md` é conferido antes do arquivamento.

## Migration Plan

Não se aplica — não há sistema em produção nem dados a migrar. Reversão é `git clean` das pastas de projeto criadas.

### 9. `GEF.Tests` como projeto único

Um só projeto de teste, com pastas internas espelhando as camadas (`Domain/`, `Application/`, `Infrastructure/`). É o que o `CLAUDE.md` define e o que cabe no tamanho do projeto.

**Alternativa considerada**: um projeto por camada (`GEF.Domain.Tests`, `GEF.Application.Tests`, …). Dá isolamento melhor de dependências de teste, mas quadruplica arquivos de projeto e tempo de restore em troca de benefício que só aparece em solução grande.

### 10. `MainWindow` mantém o nome do template

A regra de português do `CLAUDE.md` vale para **código de negócio** — entidades, casos de uso, serviços, ViewModels. Os arquivos gerados pelo template WPF (`App.xaml`, `MainWindow.xaml`) ficam com o nome original: são artefatos de infraestrutura da plataforma, reconhecíveis por qualquer desenvolvedor .NET, e renomear obriga a mexer no `StartupUri` do `App.xaml` sem ganho real.

Views e ViewModels criados de agora em diante seguem o português (`ContasView`, `ContasViewModel`).

## Open Questions

Nenhuma. As três pendências (arquivo `stylecop.json`, projeto de teste único e nome da janela inicial) foram decididas e estão registradas nas seções 3, 9 e 10.
