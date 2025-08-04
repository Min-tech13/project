FROM nginx:alpine

# Копируем 
COPY index.html /usr/share/nginx/html/

# Открываем порт 80
EXPOSE 80

# Запускаем Nginx
CMD ["nginx", "-g", "daemon off;"]