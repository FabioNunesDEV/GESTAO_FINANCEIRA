---
name: api-transicao
description: Catalogo de API e web service leitura e gravação de informações de transição. Usar quando necessário ler ou gravar informações de transição. Como as id´s podem mudar em cada ambiente/instnacia o melhor é usar a chave integração pois essa informação não deve ser diferente.
---


# Skill: Processo Designer — Transição

## Descrição

APIs do módulo **Modelador de Processos (ProcessoDesignerUI)** responsáveis por carregar e salvar as propriedades de uma **Transição**, incluindo Script, Decisão e Mensagem de Confirmação.

---

## Endpoints

### 1. Carregar Transição

Carrega todos os dados de uma transição ao abrir o modal "Propriedades da Transição".

```
GET /api/v2/processos-designer/processo-versao/{processoVersaoId}/transicao/{transicaoId}
```

**Path Parameters**

| Parâmetro | Tipo | Descrição |
|---|---|---|
| `processoVersaoId` | integer | ID da versão do processo |
| `transicaoId` | integer | ID da transição |

**Response** — retorna o objeto completo da transição, incluindo o campo `Script`.

---

### 2. Salvar Transição (Script, Decisão e demais propriedades)

Salva todas as propriedades da transição ao clicar no botão **SALVAR** do modal.

```
PUT /api/v2/processos-designer/transicao
```

**Content-Type:** `application/x-www-form-urlencoded`

**Body Parameters**

| Campo | Tipo | Obrigatório | Descrição |
|---|---|---|---|
| `Id` | integer | ✅ | ID da transição |
| `ProcessoVersaoId` | integer | ✅ | ID da versão do processo |
| `Nome` | string | ✅ | Nome da transição |
| `IntegracaoChave` | string | ✅ | Chave de integração da transição |
| `Script` | string | ✅ | Conteúdo completo do script |
| `Condicao` | string | ❌ | Expressão de decisão (aba Decisão) |
| `Padrao` | integer | ✅ | `0` = não padrão, `1` = transição padrão |
| `ExibirNome` | boolean | ✅ | Exibir nome no diagrama |
| `Ordem` | integer | ✅ | Ordem de exibição |
| `Cor` | string | ❌ | Cor da seta no diagrama (hex) |
| `CssClass` | string | ❌ | Classe CSS personalizada |
| `Tipo` | integer | ✅ | Tipo da transição (`1` = padrão) |
| `ExibeTimeout` | boolean | ✅ | Exibir transição como timeout |

---

### 3. Carregar Mensagem de Confirmação

Carrega os dados da aba "Mensagem de Confirmação" ao abrir o modal.

```
GET /api/v2/processos-designer/processo-versao/{processoVersaoId}/transicao/{transicaoId}/transicao-confirmacao
```

**Path Parameters**

| Parâmetro | Tipo | Descrição |
|---|---|---|
| `processoVersaoId` | integer | ID da versão do processo |
| `transicaoId` | integer | ID da transição |

---

### 4. Salvar Mensagem de Confirmação

```
POST /api/v2/processos-designer/transicao-confirmacao
```

**Content-Type:** `application/x-www-form-urlencoded`

**Body Parameters**

| Campo | Tipo | Descrição |
|---|---|---|
| `Id` | integer | ID da confirmação (0 se nova) |
| `ProcessoVersaoId` | integer | ID da versão do processo |
| `ProcTransicaoId` | integer | ID da transição pai |
| `Ativo` | boolean | Ativa/desativa a mensagem de confirmação |
| `MensagemConfirmacao` | string | Texto da mensagem exibida ao usuário |
| `TextoOk` | string | Label do botão de confirmação |
| `CorOk` | string | Cor do botão de confirmação (hex) |
| `TextoCancelar` | string | Label do botão de cancelar |
| `CorCancelar` | string | Cor do botão de cancelar (hex) |

---

### 5. Carregar Campos da Confirmação

Carrega os campos disponíveis para a mensagem de confirmação.

```
GET /api/v2/processos-designer/tipo-documental/{tipoDocumentalId}/transicao-confirmacao/{confirmacaoId}/transicao-confirmacao-campos
```

```
GET /api/v2/processos-designer/tipo-documental/{tipoDocumentalId}/transicao-confirmacao/{confirmacaoId}/tipo-documental-campos
```

**Path Parameters**

| Parâmetro | Tipo | Descrição |
|---|---|---|
| `tipoDocumentalId` | integer | ID do tipo documental |
| `confirmacaoId` | integer | ID da transição-confirmação |

---

### 6. Salvar Campos da Confirmação

```
POST /api/v2/processos-designer/transicao-confirmacao/{confirmacaoId}/campos
```

**Content-Type:** `application/x-www-form-urlencoded`

**Body:** Array com os IDs dos campos selecionados.

---

## Fluxo de Uso

### Abrir modal "Propriedades da Transição"

1. `GET .../transicao/{transicaoId}` — carrega dados gerais + script
2. `GET .../transicao/{transicaoId}/transicao-confirmacao` — carrega aba confirmação
3. `GET .../transicao-confirmacao/{id}/transicao-confirmacao-campos` — carrega campos configurados
4. `GET .../transicao-confirmacao/{id}/tipo-documental-campos` — carrega campos disponíveis

### Clicar em SALVAR

1. `PUT /api/v2/processos-designer/transicao` — salva dados gerais + script + decisão
2. `POST /api/v2/processos-designer/transicao-confirmacao` — salva mensagem de confirmação
3. `POST /api/v2/processos-designer/transicao-confirmacao/{id}/campos` — salva campos

---

## Notas

- O campo `Script` **não possui endpoint próprio** — é salvo junto com os demais dados da transição via `PUT /api/v2/processos-designer/transicao`.
- A autenticação é feita via sessão. O sistema verifica `GET /api/v2/usuario/sessao-ativa/{uuid}` antes de cada chamada principal.
- O `processoVersaoId` é passado na URL da página: `ProcessoDesignerUI.aspx?processoVersaoId={id}`.
