Autor: Rafael Guedes
## Login na sua VPS

Para fazer login na sua VPS, utilize SSH:

```bash
ssh your_VPS_user@VPS_IP #por padrão a hostinger usa root@vps_ip
```
## Passo 1: Atualização do Sistema

Execute os seguintes comandos para atualizar e fazer update e upgrade do seu sistema VPS:
```
sudo apt update
sudo apt upgrade
```
## Passo 2: Instalação de Pacotes Necessários

Instale o ambiente virtual Python 3 e outras dependências
```
sudo apt install python3-venv
mkdir /home/myproject
cd /home/myproject
python3 -m venv myprojectenv
source myprojectenv/bin/activate
pip install gunicorn
```
## Passo 4: Configuração do Projeto Django

Clone ou transfira sua aplicação Django para a pasta `/home/myproject`.
os arquivos do projeto tem que estar na pasta myprojects transfira todo conteúdo
dela com o seguinte comando entre na sua pasta do projeto com 
```
cd /home/seu_projeto/
sudo mv * /home/myproject 
```

Após esse comando os aquivos estara na pasta myprojects.
## Passo 5: Testando o Gunicorn

Teste se o Gunicorn está funcionando corretamente com a sua aplicação Django:
ante de entrar e fazer o teste com o gunicorn de permicões usando esses comando 

```
sudo /home/chmod -R 777 myproject
```
Agora teste o App
```
cd ~/myproject
gunicorn --bind ip_do_vps:8000 <nome_da_pasta_com_o_arquivo_wsgi>.wsgi:application
os <> é só simbolico nao inclua eles!!!!
```

Acesse `http://is_do_vps:8000` no navegador para verificar se tudo está funcionando.
de ctrl + c para finalizar depois que testar

## Passo 6: Configuração do Nginx

Instale e configure o Nginx como um servidor reverso para o Gunicorn:
```
sudo apt install nginx
```

Crie um arquivo de configuração para o seu domínio:
```bash
sudo nano /etc/nginx/sites-available/solucaoeletrica.live
```

Adicione o seguinte conteúdo ao arquivo, ajustando os caminhos e nomes conforme necessário:

```
server {
    listen 80;
    server_name nome_do_seu_dominio.com www.<nome_do_seu_dominio>.com;

    location = /favicon.ico { access_log off; log_not_found off; }
    location /static/ {
        root /home/myproject/;
    }

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/myproject/myproject.sock;
    }
}
```

Habilite o arquivo de configuração:
```
sudo ln -s /etc/nginx/sites-available/solucaoeletrica.live /etc/nginx/sites-enabled
```

Delete os arquivos default das pasta 
```
sudo rm -fr /etc/nginx/sites-available/default
sudo rm -fr /etc/nginx/sites-enable/default
```

Verifique a configuração do Nginx e reinicie:
```
sudo nginx -t
sudo systemctl restart nginx
```
## Passo 7: Configuração do Firewall

Permita o tráfego HTTP:
```
sudo ufw allow 8000
sudo ufw allow 'Nginx Full'
```
## Passo 8: Configuração do Gunicorn

Certifique-se de estar no ambiente virtual que criou anteriormente:

```
source /home/myproject/myprojectenv/bin/activate
cd /home/myproject
sudo nano /etc/systemd/system/gunicorn.service
```

Adicione o seguinte conteúdo ao arquivo, ajustando os nomes de usuário, grupos e nomes do projeto conforme necessário:

```
[Unit]
Description=gunicorn daemon for myproject
After=network.target

[Service]
User=<nome_de_usuario_que_voce_usa_para_entrar_com_ssh>
Group=www-data
WorkingDirectory=/home/myproject
ExecStart=/home/myproject/myprojectenv/bin/gunicorn --workers 3 --bind unix:/home/myproject/myproject.sock <nome_da_pasta_com_o_arquivo_wsgi>.wsgi:application

[Install]
WantedBy=multi-user.target

```

Inicie e ative o serviço Gunicorn:

```
sudo systemctl start gunicorn
sudo systemctl enable gunicorn
sudo systemctl status gunicorn
```

## Passo 9: Reiniciar o Nginx e o Gunicorn

Após fazer as configurações acima, reinicie tanto o Nginx quanto o Gunicorn para aplicar as alterações:

```
sudo systemctl daemon-reload
sudo systemctl restart nginx
sudo systemctl restart gunicorn
```
## Passo 10: Teste Final

Acesse `http://<nome_do_seu_site> no seu navegador. Sua aplicação Django deve estar acessível através do Nginx.

Lembre-se de substituir todos os valores de exemplo pelos valores específicos do seu projeto e configuração. Este guia é uma visão geral e pode precisar de ajustes para atender às suas necessidades específicas
Se tiver duvidas entre em contato que eu posso estar tentando ajudar!

Email: rafaguedes.dev@gmail.com
