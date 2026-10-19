# Portal de Compras Costalog

Primeira versão da interface para solicitações de compra da Costalog.

## O que já está disponível
- Tela de acesso com o estilo visual dos outros portais Costalog.
- Página inicial com o botão **Criar pedido de compra**.
- Formulário separado em **Cabeçalho** e **Pedido**.
- Comprovante com protocolo, data e todos os campos enviados.
- Alternância entre modo claro e escuro.
- Botões **Registrar pedido no email** presentes, ainda sem ação, conforme solicitado.

## Limitações desta versão
Esta entrega é um protótipo de interface. O formulário salva uma cópia local no navegador utilizado; isso **não** compartilha pedidos com outros computadores nem notifica a equipe de Compras em tempo real. A tela de login também é demonstrativa e ainda não valida usuários/senhas em um serviço de autenticação. Para uso corporativo real, será necessário conectar autenticação e banco de dados compartilhado (e depois implementar o fluxo de análise, aceite/recusa e notificações).

## Publicação
O workflow em `.github/workflows/pages.yml` publica o conteúdo do repositório usando GitHub Pages.


## Cotações
- Página **Cotações** para localizar cotações por status.
- Editor com as colunas do modelo Costalog: PRODUTO, VALOR, VALOR PARC., FRETE, VALOR TOTAL A VISTA, VALOR TOTAL PARC, MARCA, FORNECEDOR e LINK.
- Filtros de preço mais barato, preço médio e preço mais alto.
- Edição após salvar e exportação para Excel com cabeçalho vermelho/branco, seguindo o modelo anexado.
- Status disponíveis: Em andamento, Esperando resposta, Comprando e Finalizado.

Por padrão, os dados de cotações ficam no navegador. A sincronização entre computadores exige configurar Supabase em `assets/config.js` e criar a tabela conforme `SUPABASE_SETUP.md`. Para restringir dados ao setor de Compras com segurança, é necessário concluir também autenticação e políticas RLS; enquanto o login estiver aberto, o seletor de cargo não é uma barreira de segurança.
