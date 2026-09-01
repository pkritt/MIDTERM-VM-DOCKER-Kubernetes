# ==========================================
# Dockerfile สำหรับ Stateless Web Server (Nginx)
# ==========================================
FROM nginx:alpine
COPY ./website-files /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
