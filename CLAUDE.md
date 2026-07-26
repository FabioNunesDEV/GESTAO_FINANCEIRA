# SOLUÇÃO GESTÃO FINANCEIRA

## PROPÓSITO DA SOLUÇÃO

- Aplicativo para gestão de patrimônio, carteira, contas de cartão, contas bancárias e contas em corretoras de investimento.
- Sistema para entrada de valores, transferências e retiradas.
- Relatórios de gastos, recebimentos, transferências, rendimentos, aportes e resgates.
- Relatório de orçamento por período (mensal, anual, customizado).
- Relatório por ativo de investimento: total de aportes, total de resgates, rendimentos, preço médio e preço atualizado.
- Contas podem ter valores em outras moedas (ex.: investimentos em corretoras americanas).
- Controle de contas a pagar e a receber (valores recorrentes mensais).

## TECNOLOGIA USADA

- .NET 8
- Sistema operacional Windows 11
- Camada de UI em WPF, padrão **MVVM**
- Entity Framework Core compatível com .NET 8
- Banco de dados local (SQL Server LocalDB)
- Projeto separado para testes unitários com **xUnit**
- Clean Architecture simples (projeto pequeno)

## BIBLIOTECAS E FERRAMENTAS

Bibliotecas definidas para cada necessidade. Algumas podem não ser usadas de fato; a especificação evita decisão improvisada quando a demanda aparecer.

| Necessidade | Biblioteca / Ferramenta | Situação |
| --- | --- | --- |
| Documentação de endpoints | Swagger (Swashbuckle.AspNetCore) | Somente se houver API |
| Acesso ao banco de dados (ORM) | Entity Framework Core (compatível com .NET 8) | Confirmada |
| Mapeamento DTO ↔ Entidade | AutoMapper | Prevista |
| Padrão de código (linter) | StyleCop (StyleCop.Analyzers) | Prevista |
| Registro de logs | Serilog | Prevista |
| Envio de e-mails | SendGrid | Somente se houver notificação por e-mail |
| Jobs em background | Hangfire | Somente se houver processamento agendado |

### Serilog — destinos (sinks)

| Sink | Pacote | Uso |
| --- | --- | --- |
| Arquivo | `Serilog.Sinks.File` | Produção e desenvolvimento; arquivo diário com retenção de 30 dias |
| Debug (Output do Visual Studio) | `Serilog.Sinks.Debug` | Somente em desenvolvimento |

- Local dos arquivos: `%LOCALAPPDATA%\GEF\logs\gef-.log` (o Serilog completa a data no nome).
- Nível mínimo: `Information` em produção, `Debug` em desenvolvimento.
- Nunca registrar em log dados sensíveis (senhas, tokens, número completo de cartão ou conta).

Regras:

- Nenhuma outra biblioteca externa entra na solução sem eu aprovar antes.
- Ao adotar uma biblioteca da tabela, atualizar a coluna **Situação** para `Em uso`.

## ESTRUTURA DE PROJETOS

Nomes de projeto/camada em inglês, com prefixo `GEF`:

- `GEF.Domain` — entidades e regras de negócio, sem dependências externas.
- `GEF.Application` — casos de uso, serviços de aplicação e interfaces.
- `GEF.Infrastructure` — Entity Framework, acesso a dados e serviços externos.
- `GEF.UI` — aplicação WPF (Views + ViewModels).
- `GEF.Tests` — testes unitários (xUnit).

## MOEDA E CÂMBIO

- Contas em moeda estrangeira usam **cotação informada manualmente** pelo usuário no momento do registro.
- (Evolução futura possível: buscar cotação por API.)

## COMO FAZER

- **Código de negócio em português Brasil**: classes, métodos, variáveis, constantes e objetos.
- Os nomes de projeto/camada da arquitetura ficam em inglês (ver acima); todo o resto é em português.
- Não repetir código.
- Métodos importantes sempre com summary, principalmente quando têm argumentos.
- Toda classe com descrição do seu propósito.
- Não alongar descrições, summaries e comentários — manter o código limpo.

## COMO VOCÊ VAI AGIR

- Agir como desenvolvedor sênior .NET 8 / WPF.
- Ser um orientador, explicando como fazer o projeto sempre que eu perguntar.
- Só mexer diretamente no código quando eu solicitar explicitamente.
- Alterações de código são controladas por **OpenSpec**: se necessário, você me faz as perguntas, cria o *propose*; com meu aceite cria o *apply*; e, com os testes OK, faz o *archive*.

## ENCERRAMENTO DE SESSÃO

Quando eu pedir para encerrar a sessão, você deve:

1. Criar um arquivo de histórico na pasta `.historico`, no formato `[aaaammdd]_historico_[sequencial].md`, descrevendo todo o trabalho feito.
   - O sequencial reinicia a cada dia. Ex.: primeira sessão de hoje → `20260719_historico_1.md`; segunda sessão no mesmo dia → `20260719_historico_2.md`.
2. Fazer o commit do trabalho no Git, sempre na branch `main`, com **título e descrição em português**.
