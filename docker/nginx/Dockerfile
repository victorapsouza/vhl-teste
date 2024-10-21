# Use a imagem base do Nginx
FROM nginx:latest

# Crie os diretórios de sites se não existirem
RUN mkdir -p /etc/nginx/sites-available /etc/nginx/sites-enabled

# Copie sua configuração do Nginx para o contêiner
COPY opencms /etc/nginx/sites-available/opencms

# Crie o link simbólico para habilitar o site
RUN ln -s /etc/nginx/sites-available/opencms /etc/nginx/sites-enabled/opencms

# Exponha a porta 80
EXPOSE 80
