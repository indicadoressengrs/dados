# Painel de Indicadores — SENGE-RS

Este repositório contém o Painel de Indicadores do SENGE-RS, uma página única (`index.html`) que apresenta os indicadores das áreas de Qualificação, Recursos Humanos, Comunicação, Atendimento e Serviços e Financeiro. Para que as atualizações feitas por uma pessoa fiquem visíveis para todas as demais, o painel grava os dados num banco de dados compartilhado no Firebase (Cloud Firestore), e não mais apenas no navegador de quem digitou.

## Por que as atualizações não apareciam para os demais

Até esta versão, o botão **Salvar** gravava os dados somente no `localStorage`, que é um espaço de armazenamento interno de cada navegador. Dessa forma, a pessoa que editava via as alterações, mas os demais usuários continuavam vendo os dados originais do arquivo publicado. Além disso, quem já havia salvado algo mantinha uma cópia própria que se sobrepunha a qualquer nova versão publicada no GitHub, o que gerava divergências entre os computadores.

Com o Firebase, essa cópia passa a existir num único lugar. Assim, toda edição salva é transmitida em tempo real para quem estiver com o painel aberto, e o indicador no canto superior direito do cabeçalho informa a situação da conexão (**Sincronizado**, **Conectando…**, **Sem conexão** ou **Modo local**).

## Configuração do Firebase (feita uma única vez)

A configuração leva cerca de quinze minutos e utiliza o plano gratuito (Spark), que comporta com folga o volume de leituras e gravações deste painel. Enquanto ela não for concluída, o painel continua funcionando em **modo local**, exatamente como antes, para que não haja interrupção no uso.

### 1. Criar o projeto

Acesse <https://console.firebase.google.com>, clique em **Adicionar projeto** e dê um nome, por exemplo `painel-sengers`. O Google Analytics não é necessário e pode ser desativado, pois o painel não o utiliza.

### 2. Criar o banco de dados

No menu lateral, abra **Criar → Firestore Database** e clique em **Criar banco de dados**. Escolha a localização `southamerica-east1 (São Paulo)`, que reduz a latência para usuários no Brasil, e selecione o **modo de produção**. Em seguida, abra a aba **Regras**, substitua todo o conteúdo pelo do arquivo [`firestore.rules`](firestore.rules) deste repositório e clique em **Publicar**.

Antes de publicar, ajuste a lista de e-mails dentro da função `editor()`, pois somente as pessoas listadas ali poderão gravar dados. Essa lista é o que efetivamente protege o painel; por isso, sempre que alguém entrar ou sair da equipe de edição, é preciso alterá-la e publicar as regras novamente.

### 3. Criar as contas de quem edita

Abra **Criar → Authentication**, clique em **Vamos começar** e ative o provedor **E-mail/senha**. Depois, na aba **Usuários**, clique em **Adicionar usuário** e cadastre o e-mail e uma senha para cada pessoa autorizada, usando os mesmos e-mails informados nas regras do passo anterior.

Recomenda-se também acessar **Authentication → Configurações → Ações do usuário** e desmarcar a opção **Ativar criação (inscrição)**. Essa medida impede que terceiros criem contas por conta própria, de modo que apenas as contas cadastradas por você existam no projeto.

### 4. Ligar o painel ao projeto

Em **Configurações do projeto** (ícone de engrenagem) → **Geral** → **Seus apps**, clique no ícone **`</>`** (Web), registre o app com qualquer apelido e copie o objeto `firebaseConfig` exibido. Em seguida, cole esses valores no arquivo [`firebase-config.js`](firebase-config.js) deste repositório, preenchendo `apiKey`, `authDomain`, `projectId`, `storageBucket`, `messagingSenderId` e `appId`.

Esses valores não são secretos e podem ficar públicos no GitHub, porque a segurança é garantida pelas regras do Firestore e pelo login. Por fim, ainda no Firebase, abra **Authentication → Configurações → Domínios autorizados** e adicione o domínio onde o painel é publicado (por exemplo, `indicadoressengrs.github.io`); caso contrário, o login será recusado nesse endereço.

### 5. Transferir os dados já digitados

Os dados que foram digitados antes desta mudança estão guardados apenas no navegador de quem os digitou. Por esse motivo, a primeira pessoa a entrar no painel **a partir do computador que tem as informações mais completas** verá uma mensagem oferecendo enviá-las para a nuvem; ao confirmar, esses dados passam a ser os oficiais para todos.

Como esse envio substitui o conteúdo da nuvem, ele deve ser feito somente uma vez e no computador correto. Nos demais computadores, basta responder **Cancelar** quando a mensagem aparecer, já que essas cópias locais tendem a estar desatualizadas.

## Como editar os indicadores

Para editar, abra o módulo desejado, clique na engrenagem no cabeçalho e entre com o e-mail e a senha cadastrados no Firebase. Após o primeiro login, o navegador mantém a sessão ativa, e a engrenagem passa a abrir o formulário diretamente; para encerrar a sessão, use o botão de saída ao lado da engrenagem.

Cada formulário grava apenas a parte do painel correspondente (por exemplo, somente os dados de RH), o que evita que duas pessoas editando áreas diferentes ao mesmo tempo apaguem as alterações uma da outra. Ainda assim, se duas pessoas editarem o **mesmo** indicador simultaneamente, prevalece a última gravação.

## Estrutura dos arquivos

| Arquivo | Finalidade |
|---|---|
| `index.html` | Página do painel, com o layout, os cálculos e os dados iniciais de cada indicador. |
| `firebase-config.js` | Identificação do projeto Firebase; vazio significa modo local. |
| `firestore.rules` | Regras de segurança do banco: leitura pública e gravação apenas pelos e-mails autorizados. |

No Firestore, os dados ficam na coleção `painel`, distribuídos em três documentos: `dados` (Qualificação, RH e Comunicação), `atend` (Sócios, SENGE Saúde e NPS, além do Atendimento RD) e `fin` (Arrecadação e Exclusões PSAT).
