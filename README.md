
# Izing

Um sistema para gestão de atendimento multicanais centralizado.

Sistema possui o backend e canais baseado em:
- Whatsapp [whatsapp-web.js](https://github.com/pedroslopez/whatsapp-web.js)
- Telegram [telegraf](github.com/telegraf/telegraf)
- Instagram [instagram-private-api](https://github.com/dilame/instagram-private-api)
- Messenger [messaging-api-messenger](https://github.com/Yoctol/messaging-apis#readme)

Como escolha para o banco de dados, optamos pelo [PostgresSql 14](https://www.postgresql.org/).

No front, todas as funcionalidades são baseadas no [vue](https://vuejs.org/) e [quasar](https://quasar.dev/), com integração via REST API e Websockets.

Esse projeto tem inspiração e também é baseado no projeto fantástico [whaticket](https://github.com/canove/whaticket-community).


**IMPORTANTE**: não garantimos que a utilização desta ferramenta não irá gerar bloqueio nas contas utilizadas. São bots que em sua maioria utilizam APIs segundarias para comunicação com os fornecedores dos serviços. Use com responsabilidade!


## Screenshots
>![Doação](screenshots/Bot.gif) 
___  
>![Doação](screenshots/dashboard.gif)
___
>![Doação](screenshots/izing.gif)
___

## Principais funcionalidades

- Multíplos canais de atendimento ✅
- Multíplos usuários simultâneos por canais de atendimento ✅
- Iniciar conversa com contatos existentes (whatsapp) ✅
- Construção de Chatbot interativo ✅
- Enviar e receber mensagens ✅
- Enviar e receber mídias diversas (imagens/áudio/documentos) ✅
- Multiempresas (abordagem de base compartilhada)

## Deploy Ubuntu 20.x

```bash
  sudo apt-get install -y libgbm-dev wget unzip fontconfig locales gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils git
```

Instalar Python (precisa para o build do frontend):

```bash
sudo apt-add-repository universe
sudo apt update
sudo apt install python2-minimal
```

Instalar o pacote  build-essential:

```bash
sudo apt-get install build-essential
```

```bash
sudo apt update && sudo apt upgrade
```

Instale o node (14.x) e confirme se o comando do node -v e npm -v está disponível:

```bash
curl -fsSL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
node -v
npm -v
```

Todas as instruções abaixo assumem que você NÃO está executando como root, pois vai dar erro no puppeteer. Então vamos começar a criar um novo usuário e conceder privilégios sudo a ele:

```bash
adduser deploy
usermod -aG sudo deploy
```

Agora podemos fazer login com este novo usuário:

```bash
su deploy
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

Instalar o Postgres Docker 

```bash
docker run --name postgresql -e POSTGRES_USER=izing -e POSTGRES_PASSWORD=password -p 5432:5432 -v /data:/var/lib/postgresql/data -d postgres
```

Instalar o Redis Docker 

```bash
docker run -e TZ="America/Sao_Paulo" --name redis-izing -p 6379:6379 -d --restart=always redis:latest redis-server --appendonly yes --requirepass "password"
```

Instalar o Redis RabbitMQ 

```bash
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 --restart=always --hostname rabbitmq -v /data:/var/lib/rabbitmq rabbitmq:3-management-alpine
```
 
 Clonar este repositório:

```bash
cd ~
git clone https://github.com/w3nder/izing.io-1.git izing
```

Crie um arquivo .env de backend e preencha com as informações correta:

```bash
cp izing/backend/.env.example izing/backend/.env
nano izing/backend/.env
```

```bash
NODE_ENV=prod
BACKEND_URL=https://api.mydomain.com      #USE HTTPS HERE, WE WILL ADD SSL LATTER
FRONTEND_URL=https://myapp.mydomain.com   #USE HTTPS HERE, WE WILL ADD SSL LATTER, CORS RELATED!
PROXY_PORT=443                            #USE NGINX REVERSE PROXY PORT HERE, WE WILL CONFIGURE IT LATTER
PORT=3000

DB_HOST=localhost
DB_DIALECT=postgres
DB_USER=
DB_PASS=
DB_NAME=

IO_REDIS_SERVER=localhost
IO_REDIS_PASSWORD=password
IO_REDIS_PORT='6379'
IO_REDIS_DB_SESSION='2'

# dados do RabbitMQ / Para não utilizar, basta comentar a var
#RABBITMQ_DEFAULT_USER=admin
#RABBITMQ_DEFAULT_PASS=123456
AMQP_URL='amqp://:5672?connection_attempts=5&retry_delay=5'

# Dados para utilização do canal do facebook
FACEBOOK_APP_ID=
FACEBOOK_APP_SECRET_KEY=

ADMIN_DOMAIN=https://admin.mydomain.com


JWT_SECRET=DPHmNRZWZ4isLF9vXkMv1QabvpcA80Rc
JWT_REFRESH_SECRET=EMPehEbrAdi7s8fGSeYzqGQbV5wrjH4i
```

Executa o npm install (com a flag --force), cria o build cria as tabela e insere os registro padrão

```bash
cd izing/backend
npm install --force
npm run build
npm run db:migrate
npm run db:seed
```

Vá para a pasta frontend e instale as dependências:

```bash
cd ../frontend
npm install
cp .env.example .env
nano .env
```

```bash
URL_API='https://api.mydomain.com'
FACEBOOK_APP_ID=''
```

```bash

npm i -g @quasar/cli
quasar build -P -m pwa

```

se você não deseja instalar o quasar cli

```bash

npx quasar build -P -m pwa

```

Instale o pm2 **com sudo** e inicie o backend com ele:

```bash
sudo npm install -g pm2

cd ../backend
pm2 start dist/server.js --name izing-backend

```

Iniciar pm2 após a reinicialização:

```bash
pm2 startup ubuntu -u `YOUR_USERNAME`
```

Copie a última saída de linha do comando anterior e execute-o, é algo como:

```bash
sudo env PATH=\$PATH:/usr/bin pm2 startup ubuntu -u YOUR_USERNAME --hp /home/YOUR_USERNAM
```

Instale o nginx:

```bash
sudo apt install nginx
```

Remova o site padrão do nginx:

```bash
sudo rm /etc/nginx/sites-enabled/default
```

Crie o site para o Backend
```bash
sudo nano /etc/nginx/sites-available/izing-backend
```

```bash
server {
  server_name api.mydomain.com;

  location / {
    proxy_pass http://127.0.0.1:3000;
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
```

Crie o site para o frontend

```bash
sudo nano /etc/nginx/sites-available/izing-frontend
```

```bash
server {
  server_name myapp.mydomain.com;
  
  root /you/path/frontend/dist/pwa; # caminho da pasta dist/pwa
  
  add_header X-Frame-Options "SAMEORIGIN";
  add_header X-XSS-Protection "1; mode=block";
  add_header X-Content-Type-Options "nosniff"; 
  
  index index.html;
  charset utf-8;
  location / {
    try_files $uri $uri/ /index.html;
  }

  access_log off;

}
```

Crie os links simbólicos para habilitar os sites:

```bash
sudo ln -s /etc/nginx/sites-available/izing-backend /etc/nginx/sites-enabled
sudo ln -s /etc/nginx/sites-available/izing-frontend /etc/nginx/sites-enabled
```

Vamos alterar a configuração do nginx para aceitar 20MB de corpo nas requisições:

```bash
sudo nano /etc/nginx/nginx.conf
...

http {
  ...
  client_max_body_size 20M;  # HANDLE BIGGER UPLOADS
}

```

Teste a configuração e reinicie o nginx:

```bash
sudo nginx -t
sudo service nginx restart
```

Agora, ative o SSL (https) nos seus sites para utilizar todas as funcionalidades da aplicação como notificações e envio de mensagens áudio. Uma forma fácil de o fazer é utilizar Certbot:

Instale o certbor com snapd:

```bash
sudo snap install --classic certbot
```

Habilite SSL com nginx:

```bash
sudo certbot --nginx
```
