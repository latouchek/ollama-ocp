# Utiliser une image de base. Fedora est utilisé ici comme exemple.
FROM fedora:latest

# Mettre à jour les paquets. Installation de curl.
# Fedora utilise dnf plutôt que apt-get.
RUN dnf update -y && \
    dnf install -y curl && \
    dnf clean all

# Définir la variable d'environnement OLLAMA_HOST et HOME
ENV OLLAMA_HOST=0.0.0.0 \
    HOME=/home/ollama

# Créer le répertoire .ollama, télécharger et installer le binaire ollama,
# Créer l'utilisateur et le groupe 'ollama' et ajuster les permissions
RUN mkdir -p /home/ollama/.ollama && \
    curl -L https://ollama.com/download/ollama-linux-amd64 -o /home/ollama/ollama && \
    chmod +x /home/ollama/ollama && \
    chgrp -R 0 /home/ollama && \
    chmod -R g=u /home/ollama && \
    chown -R 1001:0 /home/ollama && \
    chmod -R 775 /home/ollama

# Utiliser l'utilisateur 1001
USER 1001

# Définir le répertoire de travail (cette ligne était commentée dans votre exemple)
WORKDIR /home/ollama

# Définir l'entrée par défaut et la commande
ENTRYPOINT ["/home/ollama/ollama"]
CMD ["serve"]
