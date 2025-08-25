# Frontend application Dockerfile
FROM nginx:1.25-alpine

# Use Tsinghua mirror for faster package installation
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.tuna.tsinghua.edu.cn/g' /etc/apk/repositories

# Install gettext for envsubst
RUN apk add --no-cache gettext

# Remove default nginx configuration
RUN rm /etc/nginx/nginx.conf

# Copy custom nginx configuration
COPY frontend/nginx.conf /etc/nginx/nginx.conf

# Copy HTML template
COPY frontend/index.html /app/index.html.template

# Create entrypoint script and set up permissions for non-root user
RUN echo '#!/bin/sh' > /app/entrypoint.sh && \
    echo '# Substitute environment variables in the HTML template' >> /app/entrypoint.sh && \
    echo 'envsubst '"'"'${POD_NAME} ${CONTAINER_IMAGE} ${NODE_NAME} ${POD_NAMESPACE}'"'"' < /app/index.html.template > /usr/share/nginx/html/index.html' >> /app/entrypoint.sh && \
    echo '# Generate API files' >> /app/entrypoint.sh && \
    echo 'cat > /usr/share/nginx/html/api-info.json << EOF' >> /app/entrypoint.sh && \
    echo '{"status":"running","pod":"${POD_NAME}","image":"${CONTAINER_IMAGE}","node":"${NODE_NAME}","namespace":"${POD_NAMESPACE}","timestamp":"$(date -Iseconds)"}' >> /app/entrypoint.sh && \
    echo 'EOF' >> /app/entrypoint.sh && \
    echo 'cat > /usr/share/nginx/html/api-pod.txt << EOF' >> /app/entrypoint.sh && \
    echo 'Pod: ${POD_NAME}' >> /app/entrypoint.sh && \
    echo 'Image: ${CONTAINER_IMAGE}' >> /app/entrypoint.sh && \
    echo 'Node: ${NODE_NAME}' >> /app/entrypoint.sh && \
    echo 'Namespace: ${POD_NAMESPACE}' >> /app/entrypoint.sh && \
    echo 'EOF' >> /app/entrypoint.sh && \
    echo '# Start nginx' >> /app/entrypoint.sh && \
    echo 'exec nginx -g "daemon off;"' >> /app/entrypoint.sh && \
    chmod +x /app/entrypoint.sh && \
    mkdir -p /tmp/client_temp /tmp/proxy_temp /tmp/fastcgi_temp /tmp/uwsgi_temp /tmp/scgi_temp && \
    chown -R nginx:nginx /tmp && \
    chown -R nginx:nginx /usr/share/nginx/html && \
    chown -R nginx:nginx /app && \
    chown nginx:nginx /etc/nginx/nginx.conf

# Set default environment variables
ENV POD_NAME="frontend-pod-unknown"
ENV CONTAINER_IMAGE="container-image-unknown"
ENV NODE_NAME="k3d-node-unknown"
ENV POD_NAMESPACE="pod-namespace-unknown"

# Switch to non-root user
USER nginx

# Expose port 8080 (non-privileged port)
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 --spider http://localhost:8080/health || exit 1

# Use the entrypoint script
ENTRYPOINT ["/app/entrypoint.sh"]
