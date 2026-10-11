# PROMPT — Site do NexusFlow ERP

Copie tudo abaixo da linha e cole na IA que vai te ajudar com o site.

---

## 0. Como você (IA) deve trabalhar

Você vai me ajudar a construir o **site do NexusFlow**, um ERP que eu desenvolvi. Este documento descreve o sistema real, o que ele faz de verdade, como ele é visualmente (cores, tipografia, componentes) e o que **não** faz. Use-o como fonte de verdade.

Regras:
1. **Não invente** funcionalidades, números, clientes, depoimentos, preços, certificações ou integrações que não estejam aqui. Se faltar informação, pergunte antes de preencher. Onde eu não souber, use placeholder claramente marcado, como `[PREÇO A DEFINIR]`.
2. **Não prometa emissão fiscal real.** O sistema controla notas internamente, mas **não** emite NF-e junto à SEFAZ (detalhes na seção 6). Isso precisa ficar honesto no site.
3. O site deve **parecer da mesma família do sistema** (seção 8): mesma paleta, mesmo azul, mesmo jeito de cards, botões e badges.
4. Antes de escrever código, me diga em poucas linhas o que entendeu e me faça as perguntas da seção 10. Depois entregue o site por etapas, para eu revisar.

---

## 1. O que é o NexusFlow

ERP (sistema de gestão empresarial) em **PHP**, voltado a **pequenos e médios negócios de comércio e varejo**. Cobre PDV/vendas, caixa, estoque, orçamentos, entregas, compras, financeiro, fiscal (controle interno), relatórios e controle de acesso por usuário.

- Nome: **NexusFlow** ("ERP NexusFlow", "Soluções em gestão" aparece no cupom).
- Logo: ícone branco de três setas circulares formando um ciclo, ao lado do nome "NexusFlow" em branco, sobre fundo azul (aprox. `#266AAB`). O arquivo é `assents/Logo.png` (imagem quadrada com fundo azul, não transparente). Se o site precisar do logo sobre fundo claro ou escuro, será preciso uma versão transparente, e eu preciso avisar isso.
- Origem: projeto pessoal meu (Lucas Amado, estudante — `[CURSO E INSTITUIÇÃO A CONFIRMAR]`), também usado como TCC. Hoje roda em produção no meu servidor caseiro. Vale posicionar o site como um produto em evolução, sem inflar a escala.
- Status: em desenvolvimento ativo.

---

## 2. Módulos e funcionalidades (o que existe de verdade)

### Vendas e atendimento
- **PDV (Frente de Caixa)** (`venda.php`), em duas colunas. À esquerda: 1) identificar cliente (por ID ou busca por nome; padrão "Consumidor Final"); 2) adicionar itens por busca (ID/nome) ou **leitor de código de barras**, com quantidade e tabela do carrinho (produto, qtd, valor unitário, subtotal, remover). À direita: resumo fixo com subtotal, desconto em R$, tipo de pedido (Balcão/Retirada ou Entrega com rua, número, bairro e taxa de entrega), forma de pagamento, parcelamento (cartão de crédito), vencimento (fiado), total final em destaque, chave "Gerar Nota Fiscal?" e botão verde **FINALIZAR VENDA**.
- Formas de pagamento: Dinheiro, Pix, Cartão de Débito, Cartão de Crédito e **Fiado** (venda a prazo, com checagem do limite de crédito do cliente). As formas são cadastráveis por empresa.
- **Cupom não fiscal térmico** (impressora de 58mm) impresso ao finalizar a venda.
- **Orçamentos**: criar proposta, aprovar, cancelar, converter em venda (com baixa de estoque) e imprimir.
- **Entregas**: painel de despacho (define entregador) e conclusão de entregas em rota.
- **Histórico de vendas e de orçamentos**: filtro por período e status, reimpressão de cupom.

### Caixa
- Abertura e fechamento de turno, com **conferência física** (dinheiro/cartão/PIX) contra o valor esperado pelo sistema. O resultado aparece como **batido** (verde), **sobra** (azul) ou **falta** (vermelho).
- **Sangria** (retirada) e **suprimento** (entrada) manuais.
- O PDV só funciona com o caixa aberto; se estiver fechado, o sistema avisa e leva para a tela de caixa.

### Cadastros
- **Clientes** (PF/PJ, endereço, limite de crédito, histórico de compras, saldo devedor do fiado).
- **Fornecedores** (dados fiscais, endereço, contato, dados bancários/PIX).
- **Estoque/produtos** (preço de custo e venda, margem, tributação NCM/CFOP/IPI/ST, fornecedor, categoria, marca, lote/validade, código de barras, estoque mínimo). Desativar produto não apaga o histórico de vendas.
- **Categorias, Marcas e Formas de Pagamento** como cadastros mestres.

### Compras e financeiro
- **Pedido de compra**: monta o pedido a partir dos produtos de um fornecedor e gera automaticamente uma conta a pagar.
- **Contas a pagar / Contas a receber**: lançamento manual, filtro por status (pendente / atrasado / pago), baixa com um clique. O fiado do PDV alimenta contas a receber.

### Fiscal (controle interno)
- Gestão interna de notas: status, filtro, cancelamento com motivo e **exportação de XMLs em lote**. Veja a limitação na seção 6.

### Relatórios gerenciais
- Curva ABC de produtos, comissão de vendedores (% configurável), estoque valorizado, ranking de clientes e DRE simplificado.

### Dashboard (Home)
- Cards de resumo: vendas do dia, produtos com estoque baixo (alerta em vermelho), orçamentos pendentes e, para gerente/admin, contas vencidas a pagar e a receber.
- Gráfico de linha com o faturamento dos últimos 7 dias e gráfico de rosca com o status dos orçamentos (Chart.js).

### Administração
- **Usuários e permissões** com 5 níveis (seção 3).
- **Meu perfil** (troca da própria senha).
- **Auditoria**: log de ações sensíveis (login, vendas, financeiro, alterações de cadastro), com filtro por usuário e ação.

---

## 3. Níveis de acesso (quem vê o quê)

| Nível | Acesso principal |
|---|---|
| **Administrador** | Tudo, inclusive formas de pagamento e auditoria |
| **Gerente** | Vendas, caixa, clientes, estoque, fornecedores, compras, financeiro, relatórios, fiscal, usuários |
| **Vendedor** | Venda, orçamento, históricos, entregas, clientes |
| **Caixa** | Caixa, históricos, entregas |
| **Estoque** | Estoque, pedido de compra, categorias e marcas |
| **Super admin** | Nível da plataforma: só vê a tela **Empresas** |

O menu mostra só as telas do papel de cada um, e cada página **também** confere o nível no servidor (esconder o menu não é a única proteção).

---

## 4. Multi-empresa (multi-tenant)

Um mesmo sistema atende várias empresas, cada uma enxergando **apenas os próprios dados**. O super admin cadastra a empresa (dados, login e senha do admin dela). O sistema cria sozinho as formas de pagamento padrão e o cliente "Consumidor Final". O login é único no sistema inteiro. **Multi-filial** (várias lojas da mesma empresa compartilhando estoque) **não** existe.

---

## 5. Tecnologia e segurança (para seções "Como funciona" e "Segurança")

**Stack:** PHP 8.x, MySQL/MariaDB via `mysqli`, HTML5/CSS3/JavaScript puro. Bibliotecas por CDN: SweetAlert2 (alertas), Select2 (buscas), Chart.js (gráficos) e Bootstrap Icons 1.11.3 (ícones). Sem framework e sem Composer; cada tela é um arquivo PHP autocontido. Roda em Apache/XAMPP/WAMP, Nginx + PHP-FPM ou no servidor embutido do PHP.

**Segurança implementada (pode ser citada no site, porque existe no código):**
- Senhas com bcrypt (`password_hash`).
- Prepared statements na maior parte do sistema.
- Token **CSRF** nos formulários e chamadas AJAX que alteram dados.
- Bloqueio de login após 5 tentativas em 1 minuto.
- Cookies de sessão `HttpOnly`, `SameSite` e `Secure` (em HTTPS).
- Nível e status do usuário revalidados no banco a cada requisição (desativar uma conta derruba o acesso na hora).
- Preço e estoque **sempre revalidados no servidor** na venda; o navegador não define valores.
- Credenciais do banco fora da pasta pública do site.
- Log de auditoria.

Não escreva "à prova de ataques", "100% seguro" ou "certificado". Escreva o que está acima.

---

## 6. Limitações (o site precisa ser honesto sobre isto)

- **Não emite NF-e/NFC-e real** junto à SEFAZ. Isso exige certificado digital e um provedor homologado (ex.: Focus NFe, PlugNotas), e está fora do escopo atual. O que existe é o controle interno de notas e a exportação de XML. Sugestão de texto: "Controle fiscal interno, com caminho para integrar um emissor homologado."
- Sem folha de pagamento/RH, sem contabilidade formal (livros fiscais, SPED) e sem multi-filial.
- O cupom é **não fiscal**.
- Ainda não é um produto comercial maduro: não há planos, preços, suporte nem contrato definidos (pergunte-me antes de criar qualquer coisa sobre isso).

---

## 7. Telas do sistema (para screenshots e mockups)

Login • Home (dashboard) • Venda (PDV) • Orçamento • Histórico de orçamentos • Histórico de vendas • Entregas • Caixa • Fechar caixa • Clientes • Estoque (lista, cadastro, edição) • Fornecedores • Pedido de compra • Categorias • Marcas • Formas de pagamento • Contas a pagar • Contas a receber • Relatórios • Notas fiscais • Usuários (lista, cadastro, edição) • Auditoria • Meu perfil • Empresas (super admin).

Se o site for usar imagens do sistema, **use dados fictícios** (nunca dados reais de clientes ou vendas) e me peça os prints.

---

## 8. Identidade visual (extraída do CSS real do sistema)

### 8.1 Paleta

| Papel | Cor | Onde aparece |
|---|---|---|
| **Azul de marca (sidebar/fundo do login)** | `#0f2942` | Menu lateral, fundo da tela de login, título "Entrar" |
| **Azul primário (ação)** | `#2563eb` | Botões primários, links, foco, títulos de seção, cards de resumo |
| **Azul primário hover** | `#1d4ed8` | Hover de botão, botão "Entrar" |
| **Azul de destaque no menu** | `#4a9eff` | Borda esquerda do item ativo da sidebar |
| **Azul claro (fundo de tag)** | `#eff6ff` / `#dbeafe` | Badges de categoria, status aberto |
| **Azul escuro de texto em tag** | `#1e40af` | Texto de badge azul |
| **Verde sucesso** | `#10b981` | Finalizar venda, entradas, valor total, ativo |
| **Verde hover** | `#059669` | Hover do botão finalizar |
| **Verde claro (tag)** | `#dcfce7` com texto `#166534` | Aprovado, caixa batido |
| **Vermelho erro/alerta** | `#ef4444` | Saídas, alerta de estoque baixo, remover |
| **Vermelho forte** | `#dc2626` | Mensagens de erro no login |
| **Vermelho do botão Sair** | `#dc3545` (hover `#bb2d3b`) | Logout |
| **Vermelho claro (tag)** | `#fee2e2` com texto `#991b1b` | Expirado, falta no caixa |
| **Âmbar aviso** | `#f59e0b` (hover `#d97706`) | Botões de atenção |
| **Fundo da página** | `#f1f5f9` | Todo o sistema |
| **Superfície (cards)** | `#ffffff` | Cards e formulários |
| **Superfície alternada** | `#f8fafc` | Cabeçalho de tabela, áreas suaves |
| **Borda padrão** | `#e2e8f0` | Cards, tabelas |
| **Borda de input** | `#cbd5e1` | Inputs e selects |
| **Texto principal** | `#0f172a` | Títulos (h2) e totais |
| **Texto de corpo** | `#334155` | Texto geral |
| **Texto secundário** | `#64748b` | Labels, cabeçalho de tabela |
| **Texto discreto** | `#94a3b8` | Dicas, inativo |
| **Fundo escuro de destaque** | `#0f172a` | Caixa do total no PDV (com valor em `#10b981`) |

Cores usadas só em atalhos da home antiga (bordas laterais dos cards de módulo): ciano `#0dcaf0`, verde `#198754`, amarelo `#ffc107`, roxo `#6f42c1`, laranja `#fd7e14`, turquesa `#20c997`. Use com parcimônia, só como acentos de categoria.

**Resumo da personalidade de cor:** base de **azul-marinho profundo + azul royal**, superfícies brancas sobre cinza-azulado muito claro, e verde/vermelho reservados para significado (dinheiro entrando/saindo, sucesso/erro). Nada de gradientes chamativos.

### 8.2 Tipografia
- Fonte do sistema: `'Segoe UI', system-ui, -apple-system, sans-serif` (a sidebar usa Arial como fallback). O cupom usa `'Courier New', monospace`.
- Para o site, recomendo **Inter** (já carregada em uma tela do sistema, pesos 400/600/700), com o stack acima como fallback. Se escolher outra, me diga o motivo.
- Hierarquia usada: h2 `1.5rem` peso 700 cor `#0f172a`; h3 de seção `0.875rem`, peso 700, **maiúsculas**, `letter-spacing: 0.05em`, cor `#2563eb`, com linha inferior; labels `12px` peso 600 cor `#64748b`; micro-rótulos `10–11px` peso 800 em **MAIÚSCULAS**; corpo `14px`; números de destaque `28–32px` peso 800.

### 8.3 Forma, sombra e espaçamento
- Cards: fundo branco, `border-radius: 12px` (cards de dashboard `15px`), borda `1px solid #e2e8f0`, sombra suave `0 4px 6px -1px rgba(0,0,0,0.05)`, padding `20–25px`.
- Inputs/botões: `border-radius: 8px`; input com padding `10–12px 14px`; **foco** com borda `#2563eb` e anel `0 0 0 3px rgba(37,99,235,0.1)`.
- Botão primário: fundo `#2563eb`, texto branco, peso 600, padding `12px 24px`, hover mais escuro com transição `0.2s`. O botão de finalizar é verde, largo, peso 800, e sobe 2px no hover.
- Badges: pílula (`border-radius: 20px`), `10–12px`, peso 700/800, com fundo claro e texto escuro da mesma família.
- Cards de resumo: coloridos e sólidos (verde = entradas, vermelho = saídas, azul = saldo), título em maiúsculas e valor `20px` peso 800.
- Stat card da home: borda esquerda de `5px` azul (vermelha quando é alerta), rótulo `11px` maiúsculo peso 800, valor `28px` peso 800.
- Tabelas: cabeçalho `#f8fafc`, texto `12px` maiúsculo `#64748b`, linhas com divisor `#f1f5f9`, linha de total em negrito.
- Sidebar: `240px`, fundo `#0f2942`, itens `14px` com ícone Bootstrap Icons à esquerda, texto branco a 75% e hover com fundo `rgba(255,255,255,0.08)`; item ativo com fundo `rgba(255,255,255,0.1)` e borda esquerda `3px #4a9eff`. Botão **Sair** vermelho no rodapé. Abaixo de 992px colapsa para `80px` só com ícones.
- Tela de login: fundo `#0f2942` inteiro, cartão branco centralizado de `380px`, raio `10px`, sombra forte `0 20px 40px rgba(0,0,0,0.25)`, logo no topo, título "Entrar", botão azul `#1d4ed8`.
- Breakpoints usados: `1100px`, `992px`, `768px`, `576px`.

### 8.4 Ícones
Bootstrap Icons 1.11.3 (traço simples). Mapeamento já usado: Home `speedometer2`, Venda `cart3`, Orçamento `file-earmark-text`, Histórico `clock-history`/`receipt`, Entregas `truck`, Caixa `cash-stack`, Clientes `people`, Estoque `box-seam`, Fornecedor `building`, Pedido de compra `clipboard-check`, Categorias `tags`, Marcas `bookmark`, Contas a pagar `arrow-up-circle`, Contas a receber `arrow-down-circle`, Relatórios `graph-up`, Formas de pagamento `credit-card`, Notas fiscais `file-earmark-ruled`, Usuários `person-badge`, Auditoria `shield-check`, Empresas `buildings`, Sair `box-arrow-right`.

### 8.5 Tom de voz
Direto, profissional e prático, em **português do Brasil**, falando de rotina de loja (balcão, caixa, estoque, fiado, entrega). Sem jargão de startup e sem exagero. Exemplo bom: "Feche o caixa conferindo dinheiro, cartão e Pix contra o que o sistema esperava." Exemplo ruim: "Revolucione sua gestão com IA de última geração."

### 8.6 Inconsistências do sistema atual (não replique)
- A `home.css` antiga usa topo `#0d6efd` (azul Bootstrap) e `Arial`, diferente do restante; a identidade correta é `#0f2942` + `#2563eb`.
- A pasta de assets se chama `assents` (grafia com erro). Isso é do sistema; o site pode usar `assets`.

---

## 9. Ideias de estrutura para o site (proponha e me deixe escolher)

1. **Hero**: proposta de valor em uma frase, botão principal ("Ver o sistema" / "Falar comigo") e um mockup do PDV.
2. **Para quem é**: comércio e varejo pequeno e médio.
3. **Módulos**: grade de cards, cada um com ícone, título e 1 linha.
4. **PDV em destaque**: carrinho, leitor de código de barras, fiado, entrega, cupom térmico.
5. **Caixa e financeiro**: conferência de caixa (batido/sobra/falta), contas a pagar e receber.
6. **Controle de acesso**: tabela dos 5 níveis.
7. **Multi-empresa**.
8. **Segurança**: lista honesta da seção 5.
9. **Tecnologia**.
10. **Limitações e roadmap**: transparência sobre o fiscal.
11. **Contato / demonstração**.
12. Rodapé com créditos (Lucas Amado, `[INSTITUIÇÃO A CONFIRMAR]`).

---

## 10. Perguntas que você deve me fazer ANTES de começar

1. Qual é o objetivo do site: **página de apresentação do produto**, **portfólio/TCC**, **site comercial para vender**, ou **área de demonstração**?
2. Quem é o público: donos de loja, banca do TCC, recrutadores?
3. Terá **preço/planos**? Se sim, quais (eu preciso definir)?
4. Terá **formulário de contato/demo**? Para qual e-mail/WhatsApp?
5. Tecnologia do site: HTML/CSS/JS estático, **PHP** (mesmo servidor do sistema), ou algo como React/Next? O sistema roda em servidor próprio com Nginx.
6. Domínio e hospedagem: já existe? O site fica junto do ERP ou separado?
7. Vou fornecer **prints reais** (com dados fictícios) ou você deve montar mockups em HTML/CSS?
8. Idiomas: só português?
9. Modo escuro: o sistema atual **não** tem; quer no site?
10. Posso citar o nome do TCC, da faculdade e meu nome?

Depois das respostas, entregue primeiro um **wireframe em texto** das seções, depois a primeira versão do código, e então iteramos.
