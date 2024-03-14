
# Utiliser une image de base. Debian est utilisé ici comme exemple.
FROM fedora:latest

# Mettre à jour les paquets. Installation de curl omise pour simplifier l'exemple.
# Fedora utilise dnf plutôt que apt-get.
RUN dnf update -y && dnf install -y curl && dnf clean all

# Définir la variable d'environnement OLLAMA_HOST
ENV OLLAMA_HOST=0.0.0.0
ENV HOME=/home/ollama
# Télécharger et installer le binaire ollama


# Créer l'utilisateur et le groupe 'ollama'
RUN mkdir -p /home/ollama/.ollama && \
    chgrp -R 0 /home/ollama && \
    chmod -R g=u /home/ollama
# Changer vers l'utilisateur 'ollama'
RUN curl -L https://ollama.com/download/ollama-linux-amd64 -o /home/ollama/ollama \
    && chmod +x /home/ollama/ollama
RUN chown -R 1001:0 /home/ollama
RUN chmod -R 775 /home/ollama/
USER 1001
# Définir le répertoire de travail
# WORKDIR /home/ollama

# Exécuter la commande avec l'utilisateur 'ollama'
ENTRYPOINT ["/home/ollama/ollama"]
CMD ["serve"]

