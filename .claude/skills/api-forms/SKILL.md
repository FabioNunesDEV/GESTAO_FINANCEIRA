---
name: api-forms
description: Catalogo de API e web service que são usados nas paginas ECM para buscar lista de formulários, salvar um formulário e buscar o código de um formulário. como os Id do mesmo formulário podem ser diferentes em diferentes instancias/ambientes das prefeituras com a solução notpaper dar sempre preferencia de identificação pela chave de integração e dai usar Id. Usar essas api sempre que for necessário ler um código de formulários e gravar no banco do notpaper um formulário alterado.
---


# APIs do Sistema de Formulários

## Visão Geral

Este documento descreve as 3 APIs identificadas na análise do módulo de Formulários do sistema ECM (NotPaper/Matriz).

---

## API 1 — Listar Formulários por Categoria

**Endpoint:** `GET /api/v2/formularios/porCategoria/{idCategoria}`

**Descrição:** Busca a lista de formulários filtrada por categoria (departamento/pasta). É chamada ao acessar a tela **Administração > Formulários** e ao selecionar um nó na árvore de categorias à esquerda.

**Parâmetros de rota:**
| Parâmetro   | Tipo    | Descrição                        |
|-------------|---------|----------------------------------|
| idCategoria | integer | ID da categoria/departamento     |

**Exemplo de chamada:**
```
GET http://localhost:8080/api/v2/formularios/porCategoria/1
```

**Resposta esperada:** Lista de formulários com campos: Id, Nome, ChaveIntegração, TipoDocumental, Padrão, CamposIndexaçãoScan.

---

## API 2 — Buscar Formulário (Design/HTML)

**Endpoint:** `POST WS/Ecm.asmx/RetornaFormularioWeb`

**Descrição:** Retorna o HTML do formulário para exibição no editor (CKEditor). É chamada ao abrir a tela de **Editor de Formulário** (ao clicar no botão de edição de um formulário). O código HTML exibido ao clicar no botão "Código-Fonte" no CKEditor é o conteúdo já carregado por esta API.

**Tipo:** WebService ASMX (POST JSON)

**Parâmetros (body JSON):**
| Parâmetro | Tipo    | Descrição             |
|-----------|---------|-----------------------|
| formId    | integer | ID do formulário      |

**Exemplo de chamada:**
```
POST http://localhost:8080/WS/Ecm.asmx/RetornaFormularioWeb
Content-Type: application/json; charset=utf-8
Authorization: Bearer {token}

{"formId": 3}
```

**Resposta esperada:** JSON com propriedade `d` contendo o HTML serializado do formulário.

---

## API 3 — Salvar Formulário

**Endpoint:** `POST WS/Ecm.asmx/SalvarFormularioWeb`

**Descrição:** Salva as alterações do formulário (design HTML). É chamada ao clicar no botão **SALVAR** na tela de Editor de Formulário.

**Tipo:** WebService ASMX (POST JSON)

**Parâmetros (body JSON):**
| Parâmetro          | Tipo    | Descrição                             |
|--------------------|---------|---------------------------------------|
| formId             | integer | ID do formulário (0 para novo)        |
| formNome           | string  | Nome do formulário                    |
| tipodocumentalId   | integer | ID do tipo documental                 |
| codigo             | string  | HTML do formulário (conteúdo CKEditor)|
| formChaveIntegracao| string  | Chave de integração                   |
| usarNoScan         | boolean | Indica se será usado no scan          |
| categoriaId        | integer | ID da categoria/departamento          |

**Exemplo de chamada:**
```
POST http://localhost:8080/WS/Ecm.asmx/SalvarFormularioWeb
Content-Type: application/json; charset=utf-8
Authorization: Bearer {token}

{
  "formId": 3,
  "formNome": "Circular Padrão",
  "tipodocumentalId": 4,
  "codigo": "<div>HTML do formulário...</div>",
  "formChaveIntegracao": "CIR_PADRAO",
  "usarNoScan": false,
  "categoriaId": 1
}
```

**Resposta esperada:** JSON com propriedade `d` contendo o resultado (true/false ou ID do formulário criado).

---

## Observações

- A API 1 usa a versão **REST v2** (`/api/v2/`), enquanto as APIs 2 e 3 são **WebServices ASMX** legados.
- Todas as chamadas requerem autenticação via token Bearer no header `Authorization`.
- O token é armazenado na sessão do usuário em `Ecm.token`.
- As APIs 2 e 3 usam o padrão de resposta `{d: ...}` típico dos ASP.NET Web Services.
- A URL base é definida via variável global `window.BASE_URL`.
