# Aplicativo Power Apps para preenchimento da lista **Controle Negócios**

Este guia cria um aplicativo (Canvas App) para inserir, editar e consultar registros da lista SharePoint:

- Site: `https://fsagrbr.sharepoint.com/sites/Barter516`
- Lista: `Controle Negócios`

> Objetivo: permitir que usuários preencham os campos da lista por formulário, com validação mínima e experiência amigável.

---

## 1) Pré-requisitos

1. Permissão de leitura/escrita na lista **Controle Negócios**.
2. Licença Power Apps (M365/Power Platform) ativa.
3. Acesso ao ambiente do Power Apps em `make.powerapps.com`.

---

## 2) Criar o app a partir da lista (modo mais rápido)

1. Acesse a lista no SharePoint.
2. No topo, clique em **Integrar > Power Apps > Criar um aplicativo**.
3. Nome sugerido: `App Controle Negócios`.
4. Clique em **Criar**.

Isso cria automaticamente um app com 3 telas:
- Browse (lista/consulta)
- Detail (detalhes)
- Edit/New (edição/cadastro)

---

## 3) Ajustes essenciais no aplicativo

## 3.1 Renomear telas e controles

- `BrowseScreen1` → `scrLista`
- `DetailScreen1` → `scrDetalhe`
- `EditScreen1` → `scrFormulario`
- `SharePointForm1` → `frmNegocio`

> Renomear facilita manutenção futura.

## 3.2 Definir título e identidade visual

No rótulo de título principal (`lblTitulo`):

```powerfx
Text = "Controle de Negócios"
```

Opcional:
- Cor do cabeçalho: `RGBA(0, 78, 152, 1)`
- Cor do texto: `Color.White`

---

## 4) Fórmulas recomendadas

## 4.1 Busca e ordenação na tela de lista

Na galeria principal (`galNegocios`) use:

```powerfx
Items = SortByColumns(
    Search(
        'Controle Negócios',
        txtBusca.Text,
        "Title"
    ),
    "ID",
    Descending
)
```

> Se sua lista usa outra coluna principal além de `Title`, substitua no `Search`.

## 4.2 Botão “Novo Negócio”

No botão de novo registro (`btnNovo`):

```powerfx
NewForm(frmNegocio);
Navigate(scrFormulario, ScreenTransition.Fade)
```

## 4.3 Clique em item da galeria para editar

No `OnSelect` do item da galeria:

```powerfx
Set(varRegistroSelecionado, ThisItem);
EditForm(frmNegocio);
Navigate(scrFormulario, ScreenTransition.Fade)
```

No `Item` do formulário:

```powerfx
Item = varRegistroSelecionado
```

## 4.4 Salvar formulário

No botão salvar (`btnSalvar`):

```powerfx
SubmitForm(frmNegocio)
```

No `OnSuccess` do formulário (`frmNegocio`):

```powerfx
Notify("Registro salvo com sucesso!", NotificationType.Success);
Back()
```

No `OnFailure` do formulário:

```powerfx
Notify("Erro ao salvar: " & frmNegocio.Error, NotificationType.Error)
```

## 4.5 Cancelar edição

No botão cancelar (`btnCancelar`):

```powerfx
ResetForm(frmNegocio);
Back()
```

---

## 5) Campos obrigatórios e validação

No painel de campos do formulário (`frmNegocio`):

1. Marque como **Obrigatório** os campos críticos do processo.
2. Reordene os campos conforme o fluxo do usuário.
3. Oculte colunas técnicas que não devem ser preenchidas manualmente.

Exemplo para bloquear botão salvar quando inválido:

```powerfx
DisplayMode = If(frmNegocio.Valid, DisplayMode.Edit, DisplayMode.Disabled)
```

---

## 6) Regras de negócio úteis (opcionais)

## 6.1 Preencher usuário automaticamente

Se houver coluna de pessoa (ex.: `Solicitante`), no cartão correspondente:

```powerfx
DefaultSelectedItems = [
    {
        '@odata.type': "#Microsoft.Azure.Connectors.SharePoint.SPListExpandedUser",
        Claims: "i:0#.f|membership|" & Lower(User().Email),
        DisplayName: User().FullName,
        Email: User().Email
    }
]
```

## 6.2 Salvar data atual automaticamente

Para coluna de data (ex.: `DataCadastro`):

```powerfx
DefaultDate = Today()
```

## 6.3 Exibir apenas registros do usuário logado (se necessário)

Na galeria:

```powerfx
Items = Filter(
    'Controle Negócios',
    Lower(Author.Email) = Lower(User().Email)
)
```

---

## 7) Publicar e compartilhar

1. Clique em **Arquivo > Salvar**.
2. Clique em **Publicar > Publicar esta versão**.
3. Clique em **Compartilhar** e adicione usuários/grupos.
4. Garanta que os usuários também tenham permissão na lista SharePoint.

---

## 8) Checklist de testes

- [ ] Criar novo registro.
- [ ] Editar registro existente.
- [ ] Validar campos obrigatórios.
- [ ] Confirmar mensagem de sucesso/erro.
- [ ] Verificar permissões de usuários finais.

---

## 9) Próximos passos recomendados

1. Adicionar filtros por status, data e responsável.
2. Criar painel com indicadores (total, em andamento, concluído).
3. Integrar Power Automate para aprovação/notificação por e-mail/Teams.
4. Versionar app e documentar mudanças por release.

