# Documentação: Sistema de Login e Segurança Avançada (tasIA)

Este documento registra todo o passo a passo da implementação do sistema de login e da arquitetura de segurança do projeto **tasIA**, hospedado no GitHub Pages e conectado ao Firebase.

## 1. Problema de Cache do GitHub Pages
**Situação Inicial:** Ao realizar o login com sucesso, o redirecionamento para a página inicial (`tasia-landing-standalone.html`) carregava uma versão antiga (em cache) que não reconhecia a sessão do Firebase, fazendo parecer que o usuário havia sido deslogado.
**Solução Aplicada:** Modificamos o redirecionamento no JavaScript do arquivo `login.html` para incluir um "Bypass de Cache" automático usando um Timestamp dinâmico. 
- *Código adicionado:* `window.location.href = "tasia-landing-standalone.html?v=" + new Date().getTime();`

## 2. Limpeza Visual (Apenas Google Auth)
**Situação Inicial:** A tela de login possuía um formulário para e-mail e senha e um botão secundário do Google.
**Solução Aplicada:** 
- Removemos todo o formulário HTML de e-mail e senha.
- Excluímos a função JavaScript `window.loginEmail`.
- A tela agora possui um design limpo e minimalista, contendo exclusivamente a opção **"Entrar com Google"**.

## 3. Lista VIP de E-mails (Autorização com Firestore)
**Objetivo:** Permitir que apenas contas específicas do Google pudessem acessar o sistema.
**Solução Aplicada:**
1. **Criação do Banco de Dados:** Criamos um banco de dados no **Firestore Database** pelo Firebase Console.
2. **Estrutura:** Criamos uma coleção chamada `emails_autorizados`. Para autorizar um e-mail, adicionamos um novo documento onde o **ID do Documento** é o próprio e-mail (ex: `tassfersilva2026@gmail.com`).
3. **Integração no Código:** O arquivo `login.html` foi atualizado para consultar esse banco de dados logo após a pessoa se autenticar no Google.
   - Se o e-mail existir no banco ➔ O acesso é liberado e o usuário é redirecionado.
   - Se o e-mail não existir ➔ O Firebase desloga o usuário na mesma hora e bloqueia a passagem.

## 4. Alerta de Erro Customizado (UI)
**Situação Inicial:** E-mails bloqueados geravam um pop-up cinza feio nativo do navegador (`window.alert()`).
**Solução Aplicada:** 
- Criamos uma `div` oculta dentro do formulário de login no `login.html`.
- Configuramos um visual vermelho e moderno (fundo claro, borda vermelha, texto em negrito).
- Atualizamos o JavaScript para exibir a mensagem: *"Seu Usuario nao tem autorização para acessar a pagina, entre em contato com o administrador"* injetada diretamente na tela em caso de falha de autorização.

## 5. Blindagem do Banco de Dados (Security Rules)
**Situação Inicial:** O banco de dados do Firestore havia sido criado no "Modo de Teste", o que significa que ficaria totalmente aberto e vulnerável por 30 dias.
**Solução Aplicada:** Fomos à aba "Regras" do Firestore no Firebase Console e substituímos a regra de teste por um código definitivo:
```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /emails_autorizados/{email} {
      allow read: if true;  // Libera a leitura para o site conseguir validar o login
      allow write: if false; // Bloqueia totalmente alterações externas
    }
  }
}
```
**Resultado:** Ninguém (mesmo usando a API Key) consegue apagar, editar ou gravar novos e-mails no banco de dados. Isso só pode ser feito pelo administrador, direto no painel do Firebase.

## 6. Cadeado na Chave da API (HTTP Referrers)
**Situação Inicial:** O repositório no GitHub é público, o que significa que a API Key do Firebase está exposta no código. Se outra pessoa copiasse a chave, poderia usá-la no site dela e consumir a cota gratuita da nossa conta.
**Solução Aplicada:**
1. Acessamos o **Google Cloud Console** > APIs e Serviços > Credenciais.
2. Adicionamos uma "Restrição de Aplicativo" do tipo **Sites (HTTP Referrers)**.
3. Autorizamos **apenas** dois links a utilizarem a chave:
   - `tassfersilva2026.github.io/*` (Nosso site principal)
   - `tasia-login-ca412.firebaseapp.com/*` (O motor de login do Google / Firebase)
**Resultado:** A API Key pode permanecer pública e visível no código, pois o próprio Google rejeitará conexões que venham de sites diferentes destes dois autorizados.

---
*Documentação gerada automaticamente para o projeto tasIA.*
