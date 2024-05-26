# Utilisation d'une image officielle nginx
FROM nginx:latest

# Copier le fichier de configuration nginx personnalisé
COPY ./nginx-chart/nginx_config.conf /etc/nginx/nginx.conf

# Exposer le port 80
EXPOSE 80

# Commande pour démarrer nginx lors de l'exécution du conteneur
CMD ["nginx", "-g", "daemon off;"]
