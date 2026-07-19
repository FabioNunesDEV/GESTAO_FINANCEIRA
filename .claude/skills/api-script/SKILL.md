---
name: api-script
description: Catalogo de API e web service que são usados no modal de atividade script para leitura e gravação do código script. Também usa id para fazer a gravação. É necessário usar de forma padrão a chave integração para descobrir qual é o id´s necessário para poder usar as api´s adequadamente.
---



# Skill: Ler e Salvar o Script de uma Atividade (Processo Designer)

Documenta as APIs usadas pelo Process Designer para **carregar** e **salvar**
o codigo de script de uma atividade do tipo SCRIPT (o conteudo exibido no
editor CodeMirror dentro do modal "PROPRIEDADES - TAREFA").

> O modal em si nao importa - o que importa sao as duas APIs abaixo.

## Contexto / Identificadores

| Parametro          | Exemplo | Origem                            |
|--------------------|---------|-----------------------------------|
| processoVersaoId   | 1364    | querystring da tela / URL         |
| atividadeId        | 6       | id da atividade (tarefa SCRIPT)   |
| tipoDocumentalId   | 47      | querystring da tela               |

Base das APIs: /api/v2/processos-designer/

Autenticacao: as chamadas exigem o token de sessao do app (header/cookie da
sessao ativa, ex.: /api/v2/usuario/sessao-ativa/{guid}). Chamadas fora do
contexto autenticado retornam 401 "Requisicao sem autorizacao".

---

## 1. BUSCAR o script (ao abrir/carregar a atividade)

Endpoint:

    GET /api/v2/processos-designer/processo-versao/{processoVersaoId}/atividade/{atividadeId}

Exemplo:

    GET /api/v2/processos-designer/processo-versao/1364/atividade/6

Retorno (200): objeto da atividade. O codigo do script (conteudo do CodeMirror)
esta no campo **ScriptValidacao**. Outros campos relevantes: Id,
ProcessoVersaoId, Nome, AtividadeTipoId, PreCondicaoScript.

> E esta resposta que popula o editor CodeMirror ao abrir o modal.

---

## 2. SALVAR o script (botao SALVAR)

Endpoint:

    PUT /api/v2/processos-designer/atividade
    Content-Type: application/x-www-form-urlencoded

Formato do corpo: form-urlencoded com o modelo serializado em notacao de
colchetes atividade[Campo]. Sao ~77 campos no total. Os principais:

| Campo                         | Descricao                                  |
|-------------------------------|--------------------------------------------|
| atividade[Id]                 | id da atividade (ex.: 6)                   |
| atividade[ProcessoVersaoId]   | id da versao do processo (ex.: 1364)       |
| atividade[Nome]               | nome da atividade                          |
| atividade[AtividadeTipoId]    | tipo (6 = SCRIPT)                          |
| atividade[ScriptValidacao]    | CODIGO DO SCRIPT (conteudo do CodeMirror)  |
| atividade[PreCondicaoScript]  | script de pre-condicao (pode ser vazio)    |
| ... demais campos             | notificacoes, e-mail, permissoes etc.      |

Resposta (200): atividade salva ("Atividade salva com sucesso").

### Chamadas auxiliares disparadas junto ao SALVAR

    PUT /api/v2/processos-designer/transicao-validacao-campos
    PUT /api/v2/processos-designer/processo-versao/{processoVersaoId}/atualizar-estado/false

---

## Fluxo resumido

1. Abrir: GET .../processo-versao/{ver}/atividade/{id} -> script vem em ScriptValidacao.
2. Salvar: PUT .../atividade (form-urlencoded) com o script em atividade[ScriptValidacao].

## Observacoes de reuso

- O campo do script e ScriptValidacao tanto na leitura (JSON) quanto na
  gravacao (atividade[ScriptValidacao]).
- O salvamento envia o objeto INTEIRO da atividade, nao apenas o script - ao
  integrar, recupere a atividade via GET, altere ScriptValidacao e reenvie
  todos os campos no PUT.
- Todas as chamadas precisam do contexto de sessao autenticada do app.
