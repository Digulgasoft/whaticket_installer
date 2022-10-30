Configuraçoes minima
vps 2 GB
primeira instalação ubuntu 18.04


WhatTicket
NOTA: A nova versão do whatsapp-web.js exigia o Node 14. Atualize suas instalações para continuar usando.

Um Sistema de Tickets muito simples baseado em mensagens do WhatsApp.

O backend usa whatsapp-web.js para receber e enviar mensagens do WhatsApp, criar tickets a partir deles e armazenar tudo em um banco de dados MySQL.

Frontend é um aplicativo de bate-papo multiusuário com recursos completos, inicializado com react-create-app e Material UI, que se comunica com o backend usando API REST e Websockets. Permite interagir com contatos, tickets, enviar e receber mensagens do WhatsApp.

NOTA: não posso garantir que você não será bloqueado usando este método, embora tenha funcionado para mim. O WhatsApp não permite bots ou clientes não oficiais em sua plataforma, portanto, isso não deve ser considerado totalmente seguro.

Como funciona?
A cada nova mensagem recebida em um WhatsApp associado, um novo Ticket é criado. Então, esse ticket pode ser acessado em uma fila na página Tickets, onde você pode atribuir o ticket a você mesmo, aceitando-o, respondendo a mensagem do ticket e, eventualmente, resolvendo-o.

As mensagens subsequentes do mesmo contato serão relacionadas ao primeiro ticket aberto/pendente encontrado.

Se um contato enviar uma nova mensagem em menos de 2 horas de intervalo e não houver nenhum ticket desse contato com status pendente/aberto, o ticket fechado mais recente será reaberto, em vez de criar um novo.

Capturas de tela
    

Características
Tenha vários usuários conversando no mesmo número do WhatsApp ✅
Conecte-se a várias contas do WhatsApp e receba todas as mensagens em um só lugar ✅ 🆕
Crie e converse com novos contatos sem tocar no celular ✅
Enviar e receber mensagem ✅
Enviar mídia (imagens/áudio/documentos) ✅
Receba mídia (imagens/áudio/vídeo/documentos) ✅
Instalação e Uso (Linux Ubuntu - Desenvolvimento)
Crie o banco de dados MySQL usando o docker: Observação: altere MYSQL_DATABASE, MYSQL_PASSWORD, MYSQL_USER e MYSQL_ROOT_PASSWORD.

```bash
docker run --name whaticketdb -e MYSQL_ROOT_PASSWORD=strongpassword -e MYSQL_DATABASE=whaticket -e MYSQL_USER=whaticket -e MYSQL_PASSWORD=whaticket --restart always -p 3306:3306 -d mariadb:latest --character-set-server=utf8mb4 --collation-server=utf8mb4_bin
```

# Ou execute usando `docker-compose` como abaixo
# Antes de copiar .env.example para .env primeiro e definir as variáveis no arquivo.
```bash
docker-compose up -d mysql
```

# Para administrar este banco de dados mysql facilmente usando phpmyadmin.
# Ele será executado por padrão na porta 9000, mas pode ser alterado em .env usando `PMA_PORT`
docker-compose -f docker-compose.phpmyadmin.yaml up -d


Instale as dependências do marionetista:
```bash
sudo apt-get install -y libxshmfence-dev libgbm-dev wget unzip fontconfig locales gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils
```

Clonar este repositório
```bash
git clone https://github.com/canove/whaticket/ whaticket
```
Vá para a pasta back-end e crie o arquivo .env:
```bash
cp .env.example .env
nano .env
```
Preencha o arquivo .env com variáveis de ambiente:

```bash
NODE_ENV=DEVELOPMENT      #ajuda na depuração
BACKEND_URL=http://localhost
FRONTEND_URL=https://localhost:3000
PROXY_PORT=8080
PORT=8080

DB_HOST=                  #IP do host do banco de dados, geralmente localhost
DB_DIALECT=
DB_USER=
DB_PASS=
DB_NAME=

JWT_SECRET=3123123213123
JWT_REFRESH_SECRET=75756756756
```

Instale dependências de backend, crie aplicativos, execute migrações e sementes:

```bash
npm install
npm run build
npx sequelize db:migrate
npx sequelize db:seed:all
```

Iniciar backend:
```bash
npm start
```

Abra um segundo terminal, vá para a pasta frontend e crie o arquivo .env:
```bash
nano .env
```
Cole este comando abaixo no nano .env com sua URL
REACT_APP_BACKEND_URL = http://localhost:8080/ # O URL do aplicativo de backend configurado anteriormente.

Inicie o aplicativo frontend:
```bash
npm start
```
Vamos para http://your_server_ip:3000/signup
Crie um usuário e faça login com ele.
Na barra lateral, acesse a página Conexões e crie sua primeira conexão do WhatsApp.
Aguarde o botão QR CODE aparecer, clique nele e leia o código qr.
Feito. Todas as mensagens recebidas pelo seu número do WhatsApp sincronizado aparecerão na Lista de Ingressos.
Implantação de produção básica
=================================================================================================


USANDO UBUNTU 20.04 VPS

Todas as instruções abaixo assumem que você NÃO está executando como root, pois dará um erro no marionetista. Então, vamos começar a criar um novo usuário e conceder privilégios sudo a ele:

```bash
adduser deploy
usermod -aG sudo deploy
```
Agora podemos fazer login com este novo usuário:
```bash
su deploy
```

Você precisará de dois subdomínios encaminhados para o ip do seu VPS para seguir estas instruções. Usaremos myapp.mydomain.com para front-end e api.mydomain.com para back-end no exemplo a seguir.

Atualize todos os pacotes do sistema:
```bash
sudo apt update && sudo apt upgrade
```

Instale o nó e confirme se o comando do nó está disponível:
```bash
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v
npm -v
```

Instale o docker e adicione seu usuário ao grupo do docker:
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable"
sudo apt update
sudo apt install docker-ce
sudo systemctl status docker
sudo usermod -aG docker ${USER}
su - ${USER}
```

Crie o banco de dados MySQL usando o docker: Observação: altere MYSQL_DATABASE, MYSQL_PASSWORD, MYSQL_USER e MYSQL_ROOT_PASSWORD.
```bash
docker run --name whaticketdb -e MYSQL_ROOT_PASSWORD=strongpassword -e MYSQL_DATABASE=whaticket -e MYSQL_USER=whaticket -e MYSQL_PASSWORD=whaticket --restart always -p 3306:3306 -d mariadb:latest --character-set-server=utf8mb4 --collation-server=utf8mb4_bin
```

# Ou execute usando `docker-compose` como abaixo
# Antes de copiar .env.example para .env primeiro e definir as variáveis no arquivo.
```bash
docker-compose up -d mysql
```

# Para administrar este banco de dados mysql facilmente usando phpmyadmin.
# Ele será executado por padrão na porta 9000, mas pode ser alterado em .env usando `PMA_PORT`
```bash
docker-compose -f docker-compose.phpmyadmin.yaml up -d
```

Clone este repositório:
```bash
cd ~
git clone https://github.com/canove/whaticket whaticket
```
Meu repositorio abaixo
```bash
git clone https://github.com/Digulgasoft/whaticket_installer.git
```

Crie um arquivo .env de back-end e preencha com os detalhes:

cp whaticket/backend/.env.example whaticket/backend/.env
```bash
nano whaticket/backend/.env
```
COPIA ESTE CODIGO ABAIVO NO .ENV COM SEU DOMINIO
```bash
NODE_ENV=
BACKEND_URL=https://api.mydomain.com      #USE HTTPS AQUI, VAMOS ADICIONAR SSL DEPOIS
FRONTEND_URL=https://myapp.mydomain.com   #USE HTTPS AQUI, ADICIONAREMOS SSL DEPOIS, RELACIONADO A CORS!
PROXY_PORT=443                            #USE A PORTA DE PROXY REVERSO DO NGINX AQUI, VAMOS CONFIGURAR MAIS TARDE
PORT=8080

DB_HOST=localhost
DB_DIALECT=
DB_USER=
DB_PASS=
DB_NAME=

JWT_SECRET=3123123213123
JWT_REFRESH_SECRET=75756756756
```

Instale as dependências do marionetista:
```bash
sudo apt-get install -y libxshmfence-dev libgbm-dev wget unzip fontconfig locales gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils
```

Instale dependências de back-end, crie aplicativos, execute migrações e sementes:
```bash
cd whaticket/backend
npm install
npm run build
npx sequelize db:migrate
npx sequelize db:seed:all
```

Inicie com npm start, você deverá ver: Servidor iniciado na porta... no console. Pressione CTRL + C para sair.

Install pm2 com sudo e inicie o backend com ele:
```bash
sudo npm install -g pm2
pm2 start dist/server.js --name whaticket-backend
```

Faça o início automático do pm2 após a reinicialização:
```bash
pm2 startup ubuntu -u `YOUR_USERNAME`
```

Copie a última saída de linha do comando anterior e execute-o, é algo como:
```bash
sudo env PATH=\$PATH:/usr/bin pm2 startup ubuntu -u YOUR_USERNAME --hp /home/YOUR_USERNAM
```

Vá para a pasta frontend e instale as dependências:
```bash
cd ../frontend
npm install
```

Crie um arquivo .env de front-end e preencha-o SOMENTE com seu endereço de backend, ele deve ficar assim:

REACT_APP_BACKEND_URL = https://api.mydomain.com/

Crie um aplicativo de frontend:

```bash
npm run build
```

Inicie o frontend com pm2 e salve a lista de processos pm2 para iniciar automaticamente após a reinicialização:
```bash
pm2 start server.js --name whaticket-frontend
pm2 save
```

Para verificar se está rodando, execute a lista pm2, deve ficar assim:

deploy@ubuntu-whats:~$ pm2 list
┌─────┬─────────────────────────┬─────────────┬─────────┬─────────┬──────────┬────────┬──────┬───────────┬──────────┬──────────┬──────────┬──────────┐
│ id  │ name                    │ namespace   │ version │ mode    │ pid      │ uptime │ .    │ status    │ cpu      │ mem      │ user     │ watching │
├─────┼─────────────────────────┼─────────────┼─────────┼─────────┼──────────┼────────┼──────┼───────────┼──────────┼──────────┼──────────┼──────────┤
│ 1   │ whaticket-frontend      │ default     │ 0.1.0   │ fork    │ 179249   │ 12D    │ 0    │ online    │ 0.3%     │ 50.2mb   │ deploy   │ disabled │
│ 6   │ whaticket-backend       │ default     │ 1.0.0   │ fork    │ 179253   │ 12D    │ 15   │ online    │ 0.3%     │ 118.5mb  │ deploy   │ disabled │
└─────┴─────────────────────────┴─────────────┴─────────┴─────────┴──────────┴────────┴──────┴───────────┴──────────┴──────────┴──────────┴──────────┘
Instalar nginx:
```bash
sudo apt install nginx
```

Remova o site padrão do nginx:
```bash
sudo rm /etc/nginx/sites-enabled/default
```
Crie um novo site nginx para o aplicativo frontend:
```bash
sudo nano /etc/nginx/sites-available/whaticket-frontend
```

Edite e preencha com essas informações, alterando server_name para o seu equivalente a myapp.mydomain.com:

server {
  server_name myapp.SEU DOMINIO.com;

  location / {
    proxy_pass http://127.0.0.1:3333;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_cache_bypass $http_upgrade;
  }
}


Crie outro para a API de back-end, alterando server_name para o seu equivalente a api.mydomain.com e proxy_pass para o URL do servidor do nó de back-end localhost:
```bash
sudo cp /etc/nginx/sites-available/whaticket-frontend /etc/nginx/sites-available/whaticket-backend
sudo nano /etc/nginx/sites-available/whaticket-backend
```

server {
  server_name api.mydomain.com;

  location / {
    proxy_pass http://127.0.0.1:8080;
    ......
}


Crie links simbólicos para habilitar sites nginx:
```bash
sudo ln -s /etc/nginx/sites-available/whaticket-frontend /etc/nginx/sites-enabled
sudo ln -s /etc/nginx/sites-available/whaticket-backend /etc/nginx/sites-enabled
```

Por padrão, o nginx limita o tamanho do corpo a 1 MB, o que não é suficiente para alguns uploads de mídia. Vamos alterá-lo para 20 MB, adicionando uma nova linha ao arquivo de configuração:
```bash
sudo nano /etc/nginx/nginx.conf
```

...
http {
    ...
    client_max_body_size 20M; # LIDAR COM UPLOADS MAIORES
}


Teste a configuração do nginx e reinicie o servidor:
```bash
sudo nginx -t
sudo service nginx restart
```

Agora, ative o SSL (https) em seus sites para usar todos os recursos do aplicativo, como notificações e envio de mensagens de áudio. Uma maneira fácil de fazer isso é usando o Certbot:

Instale o certbot:
```bash
sudo snap install --classic certbot
sudo apt update
```

Habilite o SSL no nginx (preencha/aceite todas as informações necessárias):
```bash
sudo certbot --nginx
```

Usando o docker e o docker-compose
Para executar o WhaTicket usando o docker, você deve executar as seguintes etapas:

cp .env.example .env
Agora será necessário configurar o .env usando suas informações, as variáveis são as mesmas mencionadas na implantação usando o ubuntu, com exceção das configurações do mysql que não estavam no .env.

# MYSQL
MYSQL_ENGINE=                           # default: mariadb
MYSQL_VERSION=                          # default: 10.6
MYSQL_ROOT_PASSWORD=strongpassword      # change it please
MYSQL_DATABASE=whaticket
MYSQL_PORT=3306                         # default: 3306; Use esta porta para expor o servidor mysql
TZ=America/Fortaleza                    # default: America/Fortaleza; Timezone for mysql

# BACKEND
BACKEND_PORT=                           # default: 8080; mas o acesso pelo host não usa esta porta
BACKEND_SERVER_NAME=api.mydomain.com
BACKEND_URL=https://api.mydomain.com
PROXY_PORT=443
JWT_SECRET=3123123213123                # mude por favor
JWT_REFRESH_SECRET=75756756756          # mude por favor

# FRONTEND
FRONTEND_PORT=80                        # default: 3000; Use a porta 80 para expor em produção
FRONTEND_SSL_PORT=443                   # default: 3001; Use a porta 443 para expor em produção
FRONTEND_SERVER_NAME=myapp.mydomain.com
FRONTEND_URL=https://myapp.mydomain.com

# BROWSERLESS
MAX_CONCURRENT_SESSIONS=                # default: 1; Use apenas se estiver usando sem navegador

Após definir as variáveis, execute o seguinte comando:
```bash
docker-compose up -d --build
```

Na primeira execução será necessário propagar as tabelas do banco de dados usando o seguinte comando:
```bash
docker-compose exec backend npx sequelize db:seed:all
```

SSL Certificado
Para implantar o certificado ssl, adicione-o à pasta ssl/certs. Dentro dela deve haver uma pasta de backend e frontend, e cada uma delas deve conter os arquivos fullchain.pem e privkey.pem, conforme a estrutura abaixo:

.
├── certs
│   ├── backend
│   │   ├── fullchain.pem
│   │   └── privkey.pem
│   └── frontend
│       ├── fullchain.pem
│       └── privkey.pem
└── www
Para gerar os arquivos de certificado use o certbot que pode ser instalado usando o snap, usei o seguinte comando:

Nota: O container frontend que roda o nginx já está preparado para receber a solicitação feita pelo certboot para validar o certificado.

# FRONTEND
```bash
certbot certonly --cert-name backend --webroot --webroot-path ./ssl/www/ -d api.mydomain.com
```
# BACKEND
```bash
certbot certonly --cert-name frontend --webroot --webroot-path ./ssl/www/ -d myapp.mydomain.com
```

Dados de acesso
User: admin@whaticket.com Password: admin

Atualizando
O WhaTicket está em andamento e estamos adicionando novos recursos com frequência. Para atualizar sua instalação antiga e obter todos os novos recursos, você pode usar um script bash como este:

Nota: Sempre verifique o .env.example e ajuste seu arquivo .env antes de atualizar, pois alguma nova variável pode ser adicionada.
```bash
nano updateWhaticket
```


```bash
#!/bin/bash
echo "Atualizando o Whaticket, aguarde."

cd ~
cd whaticket
git pull
cd backend
npm install
rm -rf dist
npm run build
npx sequelize db:migrate
npx sequelize db:seed
cd ../frontend
npm install
rm -rf build
npm run build
pm2 restart all
echo "Atualização concluída. Apreciar!"
```

Torne-o executável e execute-o:
```bash
chmod +x updateWhaticket
./updateWhaticket
```
