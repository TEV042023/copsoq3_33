https://console.firebase.google.com/project/copsoq3/overview?utm_campaign=firebase-welcome&utm_medium=newsletter&utm_source=email

Para hospedar uma aplicação desse tipo diretamente no Firebase Hosting com integração ao Firestore e Authentication, o processo é dividido em quatro etapas: configuração do console, inserção das credenciais no HTML, definição de regras de segurança e publicação via Firebase CLI.

Etapa 1: Configurar o Projeto no Console do Firebase
Acesse o Console do Firebase e clique em Adicionar projeto.

Digite o nome do projeto (ex: copsoq3-avaliacao), desative o Google Analytics (opcional para testes rápidos) e clique em Criar projeto.

Ativar o Firestore Database:

No menu lateral esquerdo, vá em Criação (ou Build) > Firestore Database.

Clique em Criar banco de dados.

Selecione o local do servidor (para o Brasil, escolha southamerica-east1 - São Paulo).

Escolha iniciar em Modo de produção e clique em Criar.

Ativar a Autenticação (Firebase Auth):

No menu lateral, acerte em Authentication e clique em Começar.

Na aba Sign-in method (Métodos de login), ative:

Anônimo: essencial para que o participante comum grave as respostas confidencialmente sem precisar criar conta ou digitar senha.

E-mail/senha: ative apenas a primeira opção para o acesso restrito do gestor.

Na aba Users (Usuários), clique em Adicionar usuário e cadastre o e-mail e a senha do gestor que terá acesso ao painel de analytics.

Obter as credenciais da aplicação (Config):

Na página inicial do projeto, clique no ícone </> (Web) para adicionar um app web.

Dê um apelido (ex: copsoq-web) e clique em Registrar app.

O Firebase exibirá o objeto firebaseConfig. Copie esses dados.

Etapa 2: Conectar o firebaseConfig ao Código HTML
No seu arquivo HTML (index.html), o bloco <script type="module"> deve receber as credenciais geradas no console. Assegure-se de que os módulos do Firebase (App, Auth e Firestore) estão sendo importados via CDN:

HTML
<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
    import { getAuth, signInAnonymously, signInWithEmailAndPassword } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
    import { getFirestore, collection, addDoc, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

    // 1. Cole aqui o objeto fornecido pelo console do seu projeto
    const firebaseConfig = {
        apiKey: "AIzaSy...",
        authDomain: "seu-projeto.firebaseapp.com",
        projectId: "seu-projeto",
        storageBucket: "seu-projeto.firebasestorage.app",
        messagingSenderId: "1234567890",
        appId: "1:1234567890:web:abcdef",
        measurementId: "G-XXXXXXX"
    };

    // 2. Inicialização dos serviços
    const app = initializeApp(firebaseConfig);
    const auth = getAuth(app);
    const db = getFirestore(app);

    // 3. Autenticação anônima imediata para o respondente comum
    async function initAuth() {
        try {
            await signInAnonymously(auth);
        } catch (error) {
            console.error("Falha ao inicializar sessão anônima:", error);
        }
    }
    initAuth();

    // 4. Gravação da resposta ao submeter o formulário
    window.salvarResposta = async function(dadosFormulario) {
        if (!auth.currentUser) await signInAnonymously(auth);
        
        // Caminho da coleção no Firestore
        const colRef = collection(db, 'copsoq_responses');
        await addDoc(colRef, {
            ...dadosFormulario,
            userId: auth.currentUser.uid,
            timestamp: new Date().toISOString()
        });
    };

    // 5. Login real do Gestor para ler as respostas
    window.loginGestor = async function(email, senha) {
        try {
            await signInWithEmailAndPassword(auth, email, senha);
            // Usuário logado como gestor: agora tem permissão de leitura
            carregarDadosAdmin();
        } catch (e) {
            alert("Credenciais de gestor inválidas.");
        }
    };
</script>
Etapa 3: Definir as Regras de Segurança do Firestore
Para proteger o sigilo das respostas e impedir vazamento de dados, configure as regras de leitura e escrita:

No Console do Firebase, vá em Firestore Database > aba Regras (Rules).

Substitua o conteúdo pelo código abaixo (trocando o e-mail pelo cadastrado no passo 1.4):

JavaScript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /copsoq_responses/{document=**} {
      
      // Qualquer respondente (mesmo anônimo) pode enviar seu questionário
      allow create: if request.auth != null;
      
      // Apenas o e-mail oficial do gestor pode ler e gerar o analytics
      allow read: if request.auth != null && request.auth.token.email == "seu-email-gestor@orgao.gov.br";
      
      // Ninguém pode alterar ou apagar registros enviados
      allow update, delete: if false;
    }
  }
}
Clique em Publicar.

Etapa 4: Publicar o Site via Firebase Hosting (CLI)
O Firebase Hosting distribui a aplicação em servidores globais via CDN com certificado SSL (HTTPS) automático.

1. Instalar o Node.js e a CLI do Firebase
Certifique-se de ter o Node.js instalado no computador. Abra o terminal (Prompt de Comando, PowerShell ou Terminal do VS Code) e instale as ferramentas do Firebase:

Bash
npm install -g firebase-tools
2. Autenticar no Firebase
No terminal, faça login na sua conta do Google:

Bash
firebase login
Uma janela do navegador será aberta. Conceda as permissões com a mesma conta onde você criou o projeto.

3. Organizar a pasta do projeto
Crie uma pasta no seu computador para o projeto (ou entre na pasta onde seu arquivo já está):

Bash
mkdir copsoq-app
cd copsoq-app
Dentro dessa pasta, crie uma subpasta chamada public e coloque o seu arquivo HTML renomeado exatamente para index.html:

Plaintext
copsoq-app/
└── public/
    └── index.html
4. Inicializar o Firebase Hosting
Dentro da pasta copsoq-app, execute:

Bash
firebase init hosting
Responda às perguntas interativas no terminal:

Please select an option: Escolha Use an existing project (usar projeto existente).

Select a default Firebase project: Selecione o projeto que você criou no Console (ex: copsoq3-avaliacao).

What do you want to use as your public directory?: Digite public e pressione Enter.

Configure as a single-page app?: Digite N (Não).

Set up automatic builds and deploys with GitHub?: Digite N (Não).

Se perguntar File public/index.html already exists. Overwrite?: Digite N (Não sobrescrever).

Dois arquivos de configuração serão criados automaticamente: firebase.json e .firebaserc.

5. Realizar o Deploy
Execute o comando final de publicação:

Bash
firebase deploy --only hosting
O terminal fará o upload dos arquivos e exibirá o link público final:

Plaintext
✔ Deploy complete!

Project Console: https://console.firebase.google.com/project/seu-projeto/overview
Hosting URL: https://seu-projeto.web.app
O site já estará online no endereço [https://seu-projeto.web.app](https://seu-projeto.web.app) (e alternativamente em [https://seu-projeto.firebaseapp.com](https://seu-projeto.firebaseapp.com)). Todos os domínios gerados pelo Firebase Hosting já vêm automaticamente pré-autorizados nas configurações de Authentication, sem necessidade de ajustes adicionais de CORS ou domínios manuais.
