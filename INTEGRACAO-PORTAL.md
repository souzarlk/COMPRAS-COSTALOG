# Integração do Portal de Compras com o Portal Costalog

Este documento define como o repositório do Portal Costalog pode abrir a ferramenta de Compras sem criar um segundo login.

## Papel de cada aplicação

- **Portal principal:** autentica o funcionário, verifica a permissão individual da ferramenta e decide se o perfil é `ADM`, `USUARIO` ou `SEM_ACESSO`.
- **Ferramenta Compras:** recebe nome, e-mail, ID único e perfil pelo canal de integração. O usuário comum abre a área de solicitações; `ADM` abre o painel de Compras.
- **Banco de dados compartilhado:** é responsável por persistir os pedidos entre dispositivos e por validar quem pode consultar ou alterar cada registro. O perfil recebido no navegador serve para selecionar a interface, não para garantir segurança do banco.

## Permissão recomendada no cadastro de funcionários

Mantenha a permissão específica por ferramenta, separada do tipo de administrador global. Exemplo de estrutura:

```js
{
  id: "uuid-unico-do-funcionario",
  nome: "Nome do funcionário",
  email: "funcionario@costalog.com.br",
  permissoes: {
    ferramentas: {
      comprasCostalog: "ADM" // "ADM", "USUARIO" ou "SEM_ACESSO"
    }
  }
}
```

Quando a ferramenta for aberta, o portal principal deve verificar `comprasCostalog` antes de exibir ou montar o iframe. Não use o nome digitado pelo funcionário como identificação, pois nomes podem se repetir.

## Inserção da ferramenta na página principal

Exemplo de iframe. O atributo `referrerpolicy="origin"` é importante porque a ferramenta verifica a origem que a incorporou antes de aceitar mensagens:

```html
<iframe
  id="comprasCostalogFrame"
  src="https://souzarlk.github.io/COMPRAS-COSTALOG/"
  title="Portal de Compras Costalog"
  referrerpolicy="origin"
  loading="lazy"
  allow="clipboard-read; clipboard-write"
  style="width:100%; min-height:900px; border:0; border-radius:16px">
</iframe>
```

O portal principal deve enviar o perfil somente depois de autenticar a sessão e confirmar que o funcionário tem acesso à ferramenta:

```js
const frame = document.getElementById("comprasCostalogFrame");
const toolUrl = new URL(frame.src);
const permission = funcionario.permissoes?.ferramentas?.comprasCostalog;

if (!["ADM", "USUARIO"].includes(permission)) {
  // Não renderize a ferramenta se permission for SEM_ACESSO ou estiver vazia.
  mostrarAvisoSemPermissao();
} else {
  function enviarSessaoDaFerramenta() {
    frame.contentWindow.postMessage({
      type: "COSTALOG_PORTAL_SESSION",
      session: {
        role: permission,
        user: {
          id: funcionario.id,           // ID estável e único
          email: funcionario.email,
          name: funcionario.nome
        }
      }
    }, toolUrl.origin);
  }

  // O iframe avisa quando seu JavaScript está pronto.
  window.addEventListener("message", function onReady(event) {
    if (event.origin !== toolUrl.origin) return;
    if (event.source !== frame.contentWindow) return;
    if (event.data?.type !== "COSTALOG_TOOL_READY") return;
    enviarSessaoDaFerramenta();
  });
}
```

Ao terminar a sessão principal, o portal pode notificar a ferramenta incorporada:

```js
frame.contentWindow.postMessage(
  { type: "COSTALOG_PORTAL_SIGNOUT" },
  new URL(frame.src).origin
);
```

Os nomes do objeto `funcionario` no exemplo devem ser adaptados ao modelo real usado pelo repositório principal. O portal principal deve ser a única origem que determina a permissão; a ferramenta não oferece mais um seletor interno de perfil.

## O que falta antes de liberar para toda a empresa

Hoje, esta versão do front-end mantém pedidos e cotações no armazenamento local do navegador. Isso não é uma base compartilhada entre computadores e não protege os dados contra alguém que altere o JavaScript ou o armazenamento local. Antes do uso produtivo por vários funcionários:

1. Armazene os pedidos e os lançamentos em uma base compartilhada.
2. Grave em cada pedido o ID único autenticado do solicitante, além do nome para exibição.
3. Configure as políticas de segurança no servidor/banco: `USUARIO` só pode ler seus próprios pedidos; `ADM` pode acessar e administrar os pedidos de Compras conforme as permissões concedidas.
4. Valide no servidor as permissões para alterar status, registrar compras, excluir lançamentos e consultar cotações. Esconder botões ou receber `role: "ADM"` via mensagem não substitui essa validação.
5. Associe ou migre os pedidos antigos sem ID de solicitante antes de mostrá-los aos usuários; a tela "Meus pedidos" filtra por ID/e-mail, não pelo nome digitado.

A ferramenta aceita os tipos de mensagem `COSTALOG_PORTAL_SESSION` e `COSTALOG_PORTAL_SIGNOUT` somente quando a mensagem vem da janela pai e da origem indicada pelo `document.referrer`. A mensagem não deve conter senha nem credenciais de banco.

## Campos de sessão aceitos pela ferramenta

- `session.role`: `ADM` ou `USUARIO` (também reconhece `ADMIN`, `ADMINISTRADOR`, `COMPRAS`, `USER` e `SOLICITANTE`).
- `session.user.id` ou `session.user.email`: identificador único obrigatório.
- `session.user.name`: nome para exibição e preenchimento do solicitante.
